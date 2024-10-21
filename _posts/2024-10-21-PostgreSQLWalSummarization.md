---
layout: post
title:  "PostgreSQL 17 WAL Summarization"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A new interesting feature in the management of WALs.

# PostgreSQL 17 WAL Summarization

PostgreSQL adds a new cool feature in the management of the Write Ahead Logs (WALs): the **WAL summarization**.

Two settings control the WAL Summarization:
- `summarize_wal` (by default set to `off`) indicates if the summaries have to be produced;
- `wal_summary_keep_time` indicates the amount of time (usually days) to keep the summaries before proceeding to an automatic cleanup.

Documentation for these two settings can be found [in the official documentation](https://www.postgresql.org/docs/17/runtime-config-wal.html){:target="_blank"}.
Turning on `summarize_wal` makes another process appear in the list of PostgreSQL processes: the **walsummarizer**:

<br/>
<br/>
```shell
$ ps -auxw | grep postgres
postgres       1  0.0  0.1 221044 29824 ?        Ss   13:45   0:00 postgres
postgres      27 11.2  0.0  74668  6768 ?        Ss   13:45   4:09 postgres: logger
postgres      28  0.0  0.3 221312 52088 ?        Ss   13:45   0:00 postgres: checkpointer
postgres      29  0.0  0.0 221188  9080 ?        Ss   13:45   0:00 postgres: background writer
postgres      31  0.0  0.0 221164 11768 ?        Ss   13:45   0:00 postgres: walwriter
postgres      32  0.0  0.0 222608  9720 ?        Ss   13:45   0:00 postgres: autovacuum launcher
postgres      33  0.0  0.0 222616  9208 ?        Ss   13:45   0:00 postgres: logical replication launcher
postgres     289  0.0  0.0 221652  7696 ?        Ss   13:57   0:01 postgres: walsummarizer
```
<br/>
<br/>

Such process is in charge of keeping an eye on what is changed on disk, so to produce the summaries.


WAL summaries are kept in the `pg_wal` directory, under the `summaries` subdirectory, hence in a very risky zone to walk into!

<br/>
<br/>
```shell
$ ls -1 $PGDATA/pg_wal/summaries
000000010000000023001320000000002305F8D8.summary
00000001000000002305F8D80000000026000028.summary
0000000100000000260000280000000029E52AB8.summary
000000010000000029E52AB8000000002BC82690.summary
00000001000000002BC82690000000002E015A98.summary


```
<br/>
<br/>


The summaries are used to enable the very cool new feature of **incremental backups**: since version `17` the `pg_basebackup` is able to take [incremental backups](https://www.postgresql.org/docs/17/continuous-archiving.html#BACKUP-INCREMENTAL-BACKUP){:target="_blank"}.
The idea is as follows: you run a first `pg_basebackup` as usual, so to take a so called *full backup*. Then you take other backups specifying to `pg_basebackup` the `--incremental` option, passing the manifest of the previous backup. The command will try to understand what changed from the previous backup on disk and copy over only blocks that have been changed.

Before version `17` the only way to take a good incremental backup was to use tools like the excellent `pgbackrest`, that was able to do exactly that.

Summaries are used to know which blocks on disk have changed since the last backup, so to inform `pg_basebackup` about what is needed to be copied over. WAL summaries are much smaller than the WALs themselves, and therefore can be stored for a pretty much long period with regard to the WALs. In particular, in order to be able to peform an incremental backup, there must be all the summaries covering the timeframe from the previos backup to the current moment, otherwise there will be no possibility to perform an incremental backup. Hence the need for a `wal_summary_keep_time` tunable that resembles to me the old days of `wal_keep_segments`, with all the related problems and workarounds.

Incremental backups need then to be *re-assembled* into a single backup by means of a new tool called `pg_composebackup`, not discussed here.

One thing that scaries me a lot is that there is no way to automatically delete summaries once they are turned off after having been enabled. In other words, the user is required to remove *no more useful summaries* if the summarizer process is turned off. Being the summaries in a subdirectory of `pg_wal`, and being the latter such a risky place to be into, I believe a distracted user could do a great damage to the system.
