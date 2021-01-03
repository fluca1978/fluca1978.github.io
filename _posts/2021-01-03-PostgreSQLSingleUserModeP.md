---
layout: post
title:  "Single User Mode and -P flag"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to allow corrupted catalogs repair.

Single User Mode and -P flag
---

It could happen that you can no more connect to the database because an error on a catalog happens.
<br/>
PostgreSQL is rock-solid, so this usually does not happen, but in the case of disk corruption (or sometimes because of poor human behavior), the system could be not able to connect to the database because the system catalogs for that database are *bad*.
<br/>
<br/>
When the catalogs have been corrupted at the index level, there is a chance to get back your database (and data) by restoring the system catalogs. In fact, the `REINDEX` command supports the *`SYSTEM`* option that, as the name suggests, performs a reindex at the *system* level, that is against the database catalogs.
<br/>
<br/>
There is however an *egg and chicken* problem: you can reindex only the catalogs of a database you are connected to, and if you cannot connect to such database because of an index corruption what can you do?
<br/>
Luckily `postgres` (the process) allows for a `-P` flag that *P*revents the system catalog indexes to be loaded:


<br/>
<br/>
```
 -P
   Ignore system indexes when reading system tables, but still update
   the indexes when modifying the tables. This is useful when
   recovering from damaged system indexes.
```
<br/>
<br/>

Therefore the recovery can be achieved following these steps:
- shutdown the cluster and restart it in **single user mode** (see [my article about](https://fluca1978.github.io/2019/06/27/PostgreSQLSingleMode.html){:target="_blank"});
- start a backend process ignoring the system indexes, such as
``` shell
postgres --single -P -D /your/own/pgdata your_faulty_database
```
where `your_faulty_database` is the damaged database;
- issue a full system reindex with `REINDEX SYSTEM your_faulty_database;
- restart the cluster in multi-user mode and try to connect to the faulty database.

<br/>
<br/>
Why is it important to start the cluster in single user mode, therefore **tearing down any other database and process**?
Well, PostgreSQL is smart enough to prevent you to connect *directly* to an already running cluster, that is any `postgres` process is checking against the presence of a `postmaster`:

<br/>
<br/>
```shell
% sudo -u postgres postgres -D /postgres/12 -P
FATAL:  lock file "postmaster.pid" already exists
HINT:  Is another postmaster (PID 14355) running in data directory "/postgres/12"?
```
<br/>
<br/>

The conclusion is: any data corruption is a story apart and cannot be *easily* fixed, but often PostgreSQL provides all the tools you needto recover.
<br/>
And of course, once you have recovered, you should take all precautions to backup, verify and test your data!
