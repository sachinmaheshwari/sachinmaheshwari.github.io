---
layout: post
title: "D/A/Q/B: a live-capture protocol for meeting notes"
date: 2026-07-25 09:00:00 -0000
tags: [productivity, engineering, note-taking]
---

Most meeting-notes advice tells you what a *finished* note should look like. Action items need an owner and a due date, or a task "owned by the team" is owned by no one. Decisions need the reasoning attached, or you have the same debate again in three weeks. Notes should go out promptly. All true, and all useless for the actual moment you're sitting in a meeting with someone still talking and nothing on the page yet.

That's the real problem: not what a good note looks like when it's done, but what you write down while it's still happening. Here's the system I use to solve that.

## One meeting, one note, four tags

The rule is simple: one meeting gets one note. My only job during the meeting is to capture four things, tagged as they happen:

- **D** — Decision
- **A** — Action item
- **Q** — Open question
- **B** — Blocker

That's it. No polish, no full sentences, no filing into the right place. Just fragments, bullets, names, half-thoughts — whatever gets the signal onto the page before the conversation moves on.

The minimum template:

```
# <Meeting title>
Date:
People:
Project:

## Raw
D:
A:
Q:
B:

## Clean summary
Summary:
Decisions:
Actions:
Open questions:
```

A rough live note looks like this, mid-meeting, ugly on purpose:

```
D: going with postgres over dynamo
A: sachin - send updated schema to team, fri
Q: does this affect the existing migration job
B: staging db resize blocked on infra ticket
```

Nothing here is a deliverable. It's raw material for one.

## Three tiers: raw, cleanup, promote

The system has a shape to it, and it's not original — it's the same shape as Tiago Forte's CODE framework (Capture → Organize → Distill → Express) and the Zettelkasten idea of fleeting versus permanent notes, as popularized by Sönke Ahrens (the terms aren't Luhmann's own), compressed down from a whole personal-knowledge practice to the scale of a single meeting.

**Tier one: raw capture, during the meeting.** Only D/A/Q/B. Capture is deliberately non-selective here — write down what seems to matter, not a polished account of everything said.

**Tier two: 60-second cleanup, right after.** If there's a small gap after the meeting ends, spend it filling in Summary, Decisions, Actions, and Open questions — using the fields that standard meeting-notes advice actually cares about: an owner and a due date on each action, the rationale behind each decision. This is where the finished template everyone recommends actually gets built. The D/A/Q/B tags solve the harder problem of what to capture live; the 60-second window is where that raw material turns into the owner/due-date/rationale format that's genuinely useful later.

If there's no gap, a one-line recovery stub is enough — more on that below.

**Tier three: promote, later that day, only if it matters.** Move or link the note into a durable place — a Project, a People record, a Knowledge doc — but only if the meeting produced something worth keeping permanently: a real decision, a project status change, a blocker that matters, a reusable technical insight, an ownership change. Otherwise the note just stays where it is, in the daily log, and that's fine. This is Forte's "Just-In-Case to Just-In-Time" idea applied to meetings: don't organize everything upfront in case it's needed later; defer organizing until you actually know a note has lasting value. Most captured notes should be allowed to just sit and decay in the daily log. Only a minority earn a permanent home.

## Why B is the tag that matters most, and what's genuinely different here

This blog has covered Olaf Zimmermann's markup system before — a much larger tag set (20-plus tags, including `[A]` Action, `[AD]` Architectural Decision, `[D]` Discussion, `[Q]` Question, `[R]`/`[Req]` Requirement, plus editorial and urgency tags) built for structured document and architecture review, not live meetings under time pressure.

Two things are genuinely different in D/A/Q/B, and worth calling out directly rather than just noting the overlap. First, Zimmermann's set has no blocker-equivalent tag at all. Second, it has a general-purpose `[M]` "Message" catch-all for anything that's just a notable takeaway — and D/A/Q/B deliberately has no catch-all.

Both differences point at the same design choice. The standup's classic third question — "any blockers?" — maps almost directly onto B, and it's a real gap that a tag set built for architecture review never needed to fill, because architecture review doesn't have the standup's specific job of surfacing what's stopping someone from finishing. And dropping the catch-all is deliberate: without an `[M]`-style bucket, you're forced to actually decide whether something is a Decision, an Action, or a Question — or let it go — instead of dumping it into a category that stops meaning anything once you've used it fifty times.

## The recovery stub

Some meetings end and you have zero time — you're walking straight into the next one. For those, skip the cleanup entirely and write one line: the main takeaway, plus whatever needs follow-up. That's the whole recovery stub. It's not a good note. It's enough to reconstruct what mattered later, which is the actual bar.

The principle underneath all of this: a rough note captured now is better than a perfect note never written. Every part of the system — the four tags, the 60-second window, the recovery stub, promoting only when it matters — is downstream of that one idea. Optimizing for a clean note you never finish is worse than a messy one that exists.

## Honest complications

Two things I'm not going to smooth over.

First: standard 1:1 advice favors a single persistent running document across sessions, not a new note per meeting, and one-note-per-meeting doesn't fit that perfectly. What actually happens is that promoting a note to People effectively rebuilds that persistent doc anyway — just assembled incrementally from individual meeting notes instead of existing as one document from the start. It's not a clean fit, it's a workaround that gets you to roughly the same place.

Second: I'm not going to pretend this is validated. There's no study behind D/A/Q/B specifically — no case study, no adoption data, no measured outcome. It's personal practice, shaped by adjacent research and prior art (CODE, Zettelkasten, Forte's just-in-time framing, Zimmermann's markup), not a proven method. There is some cognitive-science backing for the general logic that live capture is harder than it feels — a working-memory study found writing while listening produces worse recall than reading or listening alone, though that's a word-list recall study, not a meeting study, so it's consistent with the logic rather than proof of it. Mueller and Oppenheimer's 2014 finding that verbatim note-taking hurts conceptual understanding is the other usual citation here, but a 2019 replication by Morehead, Dunlosky, and Rawson found much weaker and inconsistent results — so if it's cited at all, it should be cited as contested, not settled.

## How to start

Next meeting, open a note with the title, and write nothing but D, A, Q, and B lines as they come up. Don't summarize, don't format, don't file it anywhere. When the meeting ends, take the 60 seconds if you have them — summary, decisions, actions with owners and dates, open questions. If you don't have the 60 seconds, write one line and move on. Later that day, promote it if it earned a permanent home. Most days, most notes won't. That's fine — the ones that matter will still be there when you need them.
