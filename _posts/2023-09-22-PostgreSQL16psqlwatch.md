---
layout: post
title:  "psql \\watch improvements"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A nice addition to the command \watch in the PostgreSQL command line client.

# psql \watch improvements

`psql` is the best command line SQL client ever, and it gets improved constantly. With the new **release of PostgreSQL 16**, also `psql` get a new nice addition: *the capability to stop a `\watch` command loop after a specific amount of iterations*.

<br/>

In this article I briefly show how the new feature works.

## What is `\watch`?

The special command `\watch` is similar to the Unix command line utility `watch(1)`: it repeats a specific command (in this case, an SQL statement) at regular time intervals.
<br/>
I tend to use this, as an example, when I want to monitor some progress or some catalogs: I write the query that will produce the result I want to observe, and then use `\watch` to schedule regular repetitions of the query. For instance:

<br/>
<br/>
```sql
testdb=# SELECT * FROM pg_stat_progress_cluster;
...

testdb=# \watch 5
```
<br/>
<br/>

The above example will show me what is going on as `CLUSTER` or `VACUUM` with a refresh ratio of 5 seconds.

<br/>

One problem of the `\watch` command is that it loops forever, meaning you need to manually stop it (e.g., `CTRL-c`).
Another approach, is to raise an exception when the query has to stop. As a nasty example:


<br/>
<br/>
```sql
testdb=# WITH exit(x) AS ( SELECT count(*) FROM pg_stat_progress_cluster )
	   SELECT p.*
	   FROM pg_stat_progress_cluster p, exit e
	   WHERE 1 / e.x > 0
	   ;

```
<br/>
<br/>


The above query will raise a `division by zero` as soon as there are no more entries in the `pg_stat_progress_cluster` view, and this will in turn stop the `\watch` command:


<br/>
<br/>
```sql
testdb=# \watch 1
                                                                          Wed 20 Sep 2023 09:08:18 PM CEST (every 1s)

  pid  | datid | datname | relid |   command   |      phase       | cluster_index_relid | heap_tuples_scanned | heap_tuples_written | heap_blks_total | heap_blks_scanned | index_rebuild_count
-------+-------+---------+-------+-------------+------------------+---------------------+---------------------+---------------------+-----------------+-------------------+---------------------
 78406 | 16385 | testdb  |  2612 | VACUUM FULL | rebuilding index |                   0 |                   6 |                   6 |               1 |                 1 |                   0
(1 row)

                                                                              Wed 20 Sep 2023 09:08:19 PM CEST (every 1s)

  pid  | datid | datname | relid |   command   |          phase           | cluster_index_relid | heap_tuples_scanned | heap_tuples_written | heap_blks_total | heap_blks_scanned | index_rebuild_count
-------+-------+---------+-------+-------------+--------------------------+---------------------+---------------------+---------------------+-----------------+-------------------+---------------------
 78406 | 16385 | testdb  |  3603 | VACUUM FULL | performing final cleanup |                   0 |                 551 |                 551 |               3 |                 3 |                   1
(1 row)

ERROR:  division by zero


```
<br/>
<br/>


While the above approach is, according to me, ugly, it serves to stop `\watch` *sometime in the future*, so it could be useful to collect historical information without having a bunch of empty executions.


## The new `\watch` count option

Staring from `psql` version 16, the `\watch` command has an option to indicate after how many iterations it has to spontaneously stop. The online help of the command is now as follows:

<br/>
<br/>
```sql
testdb=> \?
General
...
  \watch [[i=]SEC] [c=N] execute query every SEC seconds, up to N times

```
<br/>
<br/>

So it is now possible to specify to execute the `\watch` statement repeated, for instance, 7 times every 2 seconds:

<br/>
<br/>
```sql
testdb=> \watch i=2 c=7
...
```
<br/>
<br/>

Note that, in the case you want to specify the iteration counts, you have to use named parameter `c`, or the command will not understand your intention.The interval parameter does not require to be named, on the other hand, therefore to summary:
- `\watch 2` is valid and will repeat the command every 2 seconds without stopping;
- `\watch 2 c=7` is valid and will repated the command every 2 seconds, stopping after 7 iterations;
- `\watch i=2 c=7` ditto;
- `\watch c=7 i=2` ditto;
- `\watch 2 7` is *not valid* because both numbers will be considered as an interval and `psql` will abort with `\watch: interval value is specified more than once`.




# Conclusions

`psql` is a command line client with a lot of features that can help interacting with PostgreSQL, and it gets improved release after release. In my experience, there is no other command line client with as much features as `psql`, and even the small addition to `\watch` makes it even more valuable.
