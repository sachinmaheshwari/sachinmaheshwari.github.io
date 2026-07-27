---
layout: post
title: "Your database is already a job queue"
date: 2026-07-24 09:00:00 -0000
tags: [databases, architecture, distributed-systems, postgresql]
---

Every distributed job scheduler design starts the same way: a master assigns work, workers execute it. Then someone points out the obvious problem: the master is a single point of failure. So the design grows a leader election scheme, usually ZooKeeper or etcd, to elect a backup master if the first one dies.

Look closely at what just happened. The single point of failure didn't go away. It moved from "the master process" to "the leader election system." You've traded a bespoke coordinator for a slightly more sophisticated bespoke coordinator, and you're now running and monitoring two distributed systems instead of one.

There's a simpler option: stop electing a master at all. Put the jobs in a table in a database you already run, and let workers pull rows directly, coordinating through the database's own locking instead of through a scheduler process. This is an old idea: it's a specific application of the [Competing Consumers pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers), where producers drop work onto a shared queue and consumer instances compete to pull and process it, with the queue guaranteeing each item goes to exactly one consumer. People call the database version "database as a queue," which isn't a formal term. The formal pattern is Competing Consumers, applied with a relational database standing in for the queue.

This doesn't eliminate the single point of failure either. It relocates it again, this time onto infrastructure you probably already replicate, back up, and page yourself about at 3am, instead of onto a scheduler process you'd have to build that same maturity into from scratch. Whether that's a good trade depends entirely on whether your database's HA story is actually better than a home-grown master's would be. For most teams running Postgres or MySQL with real replication and failover already in place, it usually is.

## The claim query, and why it's harder than it looks

The naive version of "workers pull jobs from a table" looks like this: `SELECT ... FOR UPDATE` to grab a pending row, lock it, process it. It works, and it's wrong in a specific way: `FOR UPDATE` prevents two workers from double-claiming the same row, but it does this by making every other worker wait in line behind whoever locked first. You get a convoy: one worker at a time, single file, no matter how many workers you add.

The fix, in Postgres, is [`SKIP LOCKED`](https://www.postgresql.org/docs/current/sql-select.html), added as a modifier on `FOR UPDATE` back in Postgres 9.5. Instead of blocking on a locked row, a worker just skips it and grabs the next available one. [MySQL 8.0 added the same modifier](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html) (`NOWAIT`/`SKIP LOCKED` on `SELECT ... FOR UPDATE`) for the same reason. The canonical pattern, combining a CTE with an `UPDATE`, looks like this:

```sql
WITH cte AS (
  SELECT id FROM tasks
  WHERE status = 'pending'
  ORDER BY created_at
  LIMIT 1
  FOR UPDATE SKIP LOCKED
)
UPDATE tasks SET status = 'in_progress', started_at = CURRENT_TIMESTAMP
FROM cte WHERE tasks.id = cte.id
RETURNING tasks.*;
```

Each worker runs this, and instead of queuing behind each other, they fan out across different rows. This is the mechanism, not an incidental detail: it's the difference between a job queue that scales with worker count and one that doesn't.

Two things worth knowing before you reach for it. First, [MySQL's own docs are blunt](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html) that `SKIP LOCKED` "is not suitable for general transactional work" because it gives an inconsistent view of the data. That's fine for a job queue where you only care about grabbing *a* row, not the *right* row in some strict sense, but it's not a general-purpose locking tool. There's also a documented [deadlock bug (#111637)](https://bugs.mysql.com/bug.php?id=111637) involving `SKIP LOCKED` on secondary indexes under multi-instance load, worth knowing about if you're running MySQL at scale.

Second, row locks aren't the only mechanism. [Que](https://github.com/que-rb/que), a Postgres-backed queue for Ruby, uses advisory locks instead: session-scoped, held in memory, never written to disk, and automatically released if the connection drops. [Benchmarks from the Ruby queue shootout](https://github.com/chanks/queue-shootout) put Que around 10,000 jobs/sec, roughly 20x faster than older row-lock-based Ruby queues like DelayedJob or QueueClassic in that comparison. The tradeoff isn't "advisory locks are better," it's that they behave differently under failure. A crashed connection releases an advisory lock for free, which is convenient, but advisory locks don't play well with connection poolers running in transaction mode (like pgbouncer in its default configuration), since the lock is tied to a session the pooler may hand to someone else.

## What happens when a worker dies mid-job

This is the part that actually matters, and it's the part most people wave their hands at. A worker claims a row, and then it dies: crashes, gets OOM-killed, loses network, whatever. The row is locked or marked in-progress, forever, unless something notices and takes it back.

The standard mechanism is a heartbeat and a reaper: workers periodically touch a timestamp on the row they're processing, and a background process periodically scans for rows whose timestamp hasn't moved in longer than some threshold, and reclaims them. This isn't a new idea: [Delayed::Job](https://github.com/collectiveidea/delayed_job), the Ruby queue library, has done exactly this for years, well before `SKIP LOCKED` existed. It claims jobs via `locked_at`/`locked_by` columns rather than database row locks, reads ahead in batches (5 jobs by default), and recovers stuck jobs via a `max_run_time` setting, 4 hours by default. The project's own docs put the risk plainly: "If your job takes longer than that, another computer could pick it up." [pg-boss](https://github.com/timgit/pg-boss), a Postgres-backed queue for Node, takes the same shape: it's built explicitly on `SKIP LOCKED` for locking and has an expiration/dead-letter mechanism for jobs that get abandoned mid-flight.

Here's the honest problem with all of this: a heartbeat timeout is a bet. You're betting that "no update in N seconds" means "the worker is dead," when what it actually means is "the worker hasn't updated the timestamp in N seconds." That could mean dead, or it could mean it's alive and just slow, or stuck on a long GC pause, or partitioned from the database but still merrily executing the job on the other side of a network split. Martin Kleppmann's [write-up on distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html) makes the underlying point: a lock or lease holder might still be alive and simply slow, but the system has no way to distinguish that from dead, so it stops waiting once the lease expires regardless.

This is not a problem unique to job queues built on databases. It's the same unsolved problem underneath [Kubernetes leader election](https://pkg.go.dev/k8s.io/client-go/tools/leaderelection), where a leader renews a `Lease` object periodically and loses leadership if it fails to renew within a configured window (15 seconds by default in client-go). It's the same problem underneath [Kafka consumer group rebalancing](https://kafka.apache.org/41/configuration/consumer-configs/), where consumers heartbeat to the group coordinator and get marked dead (triggering a rebalance and redistribution of their partitions) if no heartbeat arrives before `session.timeout.ms` expires, with `heartbeat.interval.ms` typically set to at most a third of that timeout so there's margin. Kubernetes, Kafka, and a reaper scanning a jobs table are all doing the same thing: picking a timeout, and treating "no signal within the timeout" as "dead." All three have the same flaw, because it's not a bug in any of them: a timeout fundamentally cannot distinguish dead from slow. Clocks drift, networks partition, processes pause; liveness checks built on timeouts are approximate by construction, not by mistake.

## Fencing tokens, and the actual safety net

If a reaper reclaims a job that wasn't actually dead, just slow, you now have two workers who both think they own it: the original worker, still plugging away, and the new one the reaper just handed the job to. The standard distributed-systems fix for this is a fencing token: a monotonically increasing number attached to each lease or claim, where every write includes the current token and the underlying storage rejects any write carrying a stale one. Applied to a jobs table, this is just optimistic concurrency at the row level: give each claim a version or generation number, and when the reaper reclaims a row it bumps that number. If the original, zombie worker finishes its work and tries to write its result, its write carries the old version number and gets rejected or ignored.

That handles the write conflict. It doesn't handle the more basic issue: the job already ran twice. The reclaimed job is, by definition, a redelivery: this pattern gives you at-least-once execution, not exactly-once, and no fencing token changes that. Since you cannot reliably tell dead from slow, the only durable answer is to stop trying to prevent double execution and make double execution harmless instead. The standard approach is an idempotency key supplied with the job, stored transactionally alongside the result of doing the work, so that if the job runs a second time, the handler can check "have I already recorded a result for this key?" and short-circuit if so. This is the actual safety net for this entire architecture. The heartbeat and the reaper are what make the system self-healing; idempotency is what makes it correct even when the heartbeat guesses wrong.

## Where it stops working

Worth saying plainly: this pattern has a ceiling, and the ecosystem doesn't pretend otherwise. [Celery's own documentation](https://docs.celeryq.dev/en/latest/faq.html) recommends a proper AMQP broker like RabbitMQ for strict reliability, though it treats a database or Redis as generally fine otherwise. Celery's recommended path for reliability is push-based, over AMQP, not polling a table.

There's data behind that caution. [DBOS's benchmarking](https://www.dbos.dev/blog/making-postgres-queues-scale) of a Postgres-backed queue found a plain, unoptimized setup tops out around 100 workflows/sec. Adding `SKIP LOCKED` with `READ COMMITTED` isolation gets you to roughly 1,000/sec. `REPEATABLE READ` actually made things worse at that concurrency, causing cascading serialization failures. With selective indexing on top, they got past 30,000/sec. A [related piece digging into the root cause](https://richyen.com/postgres/2026/05/04/postgres_job_queue.html) points at MultiXact SLRU contention: many transactions briefly referencing the same row through `SKIP LOCKED` before one of them wins, exhausting a fixed-size internal cache, compounded by table bloat and WAL overhead. The rough guidance that falls out of this: at throughput in the low hundreds of jobs per second, plain `SKIP LOCKED` is fine and simple. Beyond that, look at selective indexing, advisory locks, or a dedicated queueing system. At genuinely high throughput, reach for Redis or Kafka instead of asking your relational database to be something it isn't.

One more practical wrinkle: if every worker polls on a fixed interval, they'll periodically pile up on the database at the same moment, a thundering herd on every poll cycle. The standard mitigation is exponential backoff with full jitter, spreading the retries out instead of letting them synchronize.

None of this makes the database-as-coordinator pattern wrong. It makes it a pattern with a known operating range, like every other pattern in this space. Inside that range (moderate throughput, a database you already trust) it gets you a working job queue with no new infrastructure, a claim query you can read in ten seconds, and a failure mode (a stuck row) you can debug with `SELECT * FROM tasks WHERE status = 'in_progress'` instead of a support ticket to whoever owns your scheduler cluster. Outside that range, the honest answer is to stop pretending your database is a message broker and use one.
