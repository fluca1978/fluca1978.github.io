---
layout: post
title:  "pgenv gains a new command (and contributor!)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv
permalink: /:year/:month/:day/:title.html
---
A new command in the pgenv script.

# pgenv gains a new command (and contributor!)

[pgenv](https://github.com/theory/pgenv){:target="_blank"} , the PostgreSQL binary manager written as a Bourne Again Shell script, has gained a new command: `status`.

The idea of this command is to report the status of a selected PostgreSQL instance, mainly if it is running or not.
[Behind the scenes](https://github.com/theory/pgenv/commit/70af4d4e1de28b41e39c89927c338f23e89b4378){:target="_blank"}  the implementation exploits the `pg_ctl` command for the selected instance, stopping the execution immediatly if the user has no selected any instance.

The output of `pg_ctl` has been mangled to appear a little less verbose, in particular the `pg_ctl:` prefix has been removed.

[Brian Salehi](https://github.com/briansalehi){:target="_blank"} is the author of this patch, and hopefully a new contributor that will help improving `pgenv` again and again.

As an example, when using the new `status` command you will get something like the following:

<br/>
<br/>
```shell
% pgenv status
server is running (PID: 51503)
/usr/pgsql-16/bin/postgres "-D" "/postgres/16/data"
```
<br/>
<br/>
