---
layout: post
title:  "psql \watch now has a row limit"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A new feature introduced with PostgreSQL 17.

# psql \watch now has a row limit

I often use `\watch`, an internal command of the great text client `psql` that allows to monitor the last executed query at specific interval times.
Essentially, it works as `watch(8)` on a Unix machine.

In the last major release of PostgreSQL, the `\watch` command has gained a new interesting feature: the **minrows** limit. The idea is to make `\wwatch` to stop automatically as soon as the executed query returns less than the specified number of rows.
This is great, according to me, since I often launch `\watch` to just come back to the terminal and find out a lot of *empty executions*, needing therefore to scroll up the terminal or the log to find the last point when something did happened. With this feature, `\watch` will stop for me!

As a very trivial example, imagine to launch a well known `pgbench` test (with short time parameters for the sake of this article):

<br/>
<br/>
```shell
% pgbench -T 120 -c 4 -n -U pgbench pgbench
```
<br/>
<br/>

and on a `psql` terminal, use `\watch` waiting for the `pgench` to finish:

<br/>
<br/>
```sql
pgbench=> select query, wait_event
          from pg_stat_activity
		  where datname = current_database() and usename = 'pgbench';

pgbench=> \watch m=1
```
<br/>
<br/>


Whenever the query returns less than one row, that means no more processes are connected to the database, the `\watch` will stop:


<br/>
<br/>
```sql
pgbench=> \watch m=1

                             Wed Nov  6 07:34:20 2024 (every 2s)

                                    query                                     |  wait_event
------------------------------------------------------------------------------+---------------
 UPDATE pgbench_accounts SET abalance = abalance + 4197 WHERE aid = 9520572;  | DataFileWrite
 UPDATE pgbench_accounts SET abalance = abalance + 1973 WHERE aid = 98188924; | DataFileRead
 UPDATE pgbench_accounts SET abalance = abalance + 4479 WHERE aid = 2905019;  | DataFileWrite
 UPDATE pgbench_accounts SET abalance = abalance + 2554 WHERE aid = 17213075; | DataFileRead
(4 rows)

                              Wed Nov  6 07:34:22 2024 (every 2s)

                                     query                                     |  wait_event
-------------------------------------------------------------------------------+---------------
 BEGIN;                                                                        |
 UPDATE pgbench_accounts SET abalance = abalance + -3617 WHERE aid = 4375996;  | DataFileWrite
 UPDATE pgbench_accounts SET abalance = abalance + 1388 WHERE aid = 75850626;  | DataFileWrite
 UPDATE pgbench_accounts SET abalance = abalance + -3372 WHERE aid = 15435869; | DataFileRead
(4 rows)

Wed Nov  6 07:34:24 2024 (every 2s)

 query | wait_event
-------+------------
(0 rows)

```
<br/>
<br/>

Clerly, this has its own drawbacks: imagine a long running job pause, releasing the connection, just to come back to its activity later. If `\watch` executes the query in the pause period of time, the command will stop and you will not get any update about the resuming of the same activity.
An example of this is by monitoring `pg_stat_progress_xxx` views, for example `pg_stat_procress_autovacuum` to see when cleaning a table is performed.

However, keeping in mind a good condition (number of rows) to get `\watch` on track, and being able to make it stop automatically is a feature that really helps me in my daily activity.
