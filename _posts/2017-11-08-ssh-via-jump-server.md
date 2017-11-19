---
title: "How to do ssh via jump server?"
layout: post
date: "2017-11-07"
---

Sometimes, in order to improve security and restrict access, System Administrators can set up one intermediate machine ("jump") that should be used to access any secure machine. So to access this secure machine we have to do ssh to jump machine (or intermediate machine) and then from that machine we have to do ssh to another machine. Although its good in terms of security but it adds one more step in developers work-flow. Furthermore, it complicates scp too. As now you have to download or upload to jump server first then you can do the same operation from jump to secure machine.



Here are the steps that can help in reducing this complication. After following these instructions, the intermediate or jump machine step will become completely transparent to you.

To make it easier to understand. Let's say jump is the hostname of the machine which is the intermediate one and prod1 is the target machine.

So with that intermediate step these will be the commands that you have to run:

```shell
your-machine$ ssh jump
jump$ ssh prod1
prod1$ _
```

And for scp:

```shell
jump$ scp prod1:~/somefile.txt .
jump$ logout
your-machine$ scp jump:~/somefile.txt .
```

To make it simple, we need to edit ~/.ssh/config file. Create it if it doesn't exist

Now just add these entries in ~/.ssh/config file

```shell
Host jump
  User <user name>
  Port 22
  Hostname <ip of jump machine>

Host prod1
  Hostname <ip of prod 1>
  Port 22
  ProxyCommand ssh -o 'ForwardAgent yes' jump 'ssh-add && nc %h %p'

```

That's it!

Now we can run this command. No more jump step needed.

```shell
your-machine$ ssh prod1
prod1$ _
```

And it helps with scp too.

```shell
your-machine$ scp prod1:~/somefile.txt .
```

And the file will be downloaded on your machine without you ever worrying about jump server.
