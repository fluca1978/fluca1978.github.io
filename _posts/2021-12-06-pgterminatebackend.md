---
layout: post
title:  "kill that backend!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to kill a backend process, the right way!

# `kill` that backend!

Sometimes it happens: you need, as a DBA, to be harsh and terminate a backend, that is a user connection.
<br/>
There are two main ways to do that:
- use the operating system `kill(1)` command to, well, *kill* such process;
- use PostgreSQL administrative functions like `pg_terminate_backend()` or the more polite `pg_cancel_backend()**.

## PostgreSQL `pg_cancel_backend()` and `pg_terminate_backend()`

What is the difference between the two functions?
<br/>
Quite easy to understand: `pg_cancel_backend()` sends a `SIGINT` to the backend process, that is it *asks politely to exit*. It is the equivalent of a standard `kill -INT` against the process.
<br/>
But, what does it mean to *aks politely to exit*? It means **to cancel the current query**, that is it does not terminates the user session, rather the user interaction. That is why it is mapped to `SIGINT`, the equivalent to `CTRL-c` (interrupt by keyboard).
<br/>
On the other hand, `pg_terminate_backend()` sends a `SIGTERM` to the process, that is equivalent to `kill -TERM` and *forces brutally the process to exit*.
<br/>



## Now, Kill it!

Which method should you use?
<br/>
**If you are absolutely sure about what you are doing, you can use whatever method you want!**
<br/>
But sometimes caffeine is at a too low level in your body to do it right, **you should use the PostgreSQL way**!
There are at least two good reasons to use the [PostgreSQL administrative functions](https://www.postgresql.org/docs/14/functions-admin.html){:target="_blank"}:
- you don't need access to the server, i.e., you don't need an operating system shell;
- you will not accidentally kill another process.

<br/>
The first reason is really simple to understand, and improves security about the machine hosting PostgreSQL, at least in my opinion.
<br/>
The second reason is a little less obvious, and relies on the fact that `pg_cancel_backends()` and `pg_terminate_backend()` **act only against processes within the PostgreSQL space**, that is only processes spawn by the `postmaster`.
<br/>
Let's see this in action: imagine we select the wrong process to kill, like `174601` that is running Emacs on the server.

<br/>
<br/>
```shell
% ssh luca@miguel 'ps -aux | grep emacs'
luca      174601  1.6  4.6 320068 46584 pts/0    S+   08:40   0:04 emacs


% psql -h miguel -U postgres -c "SELECT pg_cancel_backend( 174601 );" testdb
WARNING:  PID 174601 is not a PostgreSQL server process
 pg_cancel_backend 
-------------------
 f
(1 row)



% psql -h miguel -U postgres -c "SELECT pg_terminate_backend( 174601 );" testdb
WARNING:  PID 174601 is not a PostgreSQL server process
 pg_terminate_backend 
----------------------
 f
(1 row)

```
<br/>
<br/>

As you can see, there is no way to misbehave against a non PostgreSQL process! The logs provide, of course, the very same warning message:

<br/>
<br/>
```
WARNING:  PID 174601 is not a PostgreSQL server process
```
<br/>
<br/>

Now, imagine what happened if the administrator did run something like:

<br/>
<br/>
```shell
% ssh luca@miguel 'sudo kill 1747601'
```
<br/>
<br/>

The process, in this case Emacs, would have been killed.

# Conclusions

While you can always use the well known Unix tools to interact with PostgreSQL processes, it is strongly suggested to use the PostgreSQL tools. This improves safety checks and requires less effort in keeping track of what is happening on the cluster.
