---
layout: post
title:  "To WAL or not to WAL? When unlogged becomes logged..."
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
What happens to table that are not logged into WALs when a physical replication is in place?


# To WAL or not to WAL? When unlogged becomes logged...


## Creating and populating a database to test


First of all, let's create a clean database just to keep the test environment separated from other databases:

<br/>
<br/>
```sql
testdb=# CREATE DATABASE rep_test WITH OWNER luca;
CREATE DATABASE
```
<br/>
<br/>

Now let's create and populate three tables (one temporary, one unlogged and one *normal*):

<br/>
<br/>
```sql
rep_test=> CREATE TABLE t_norm( pk int GENERATED ALWAYS AS IDENTITY,
                  t text,
                  primary key( pk ) );

rep_test=> CREATE UNLOGGED TABLE 
           t_unlogged( like t_norm including all );

rep_test=> CREATE TEMPORARY TABLE 
           t_temp( like t_norm including all );
           

rep_test=> INSERT INTO t_norm( t )
               SELECT 'Row #' || v
               FROM generate_series( 1, 1000000 ) v;
INSERT 0 1000000
Time: 4712.185 ms (00:04.712)

rep_test=> INSERT INTO t_temp( t )
            SELECT 'Row #' || v
            FROM generate_series( 1, 1000000 ) v;
INSERT 0 1000000
Time: 1789.473 ms (00:01.789)

rep_test=> INSERT INTO t_unlogged( t )
               SELECT 'Unlogged #' || v
               FROM generate_series( 1, 1000000 ) v;
INSERT 0 1000000
Time: 1746.729 ms (00:01.747)
```
<br/>
<br/>

The situation now is as follows:
<br/>


| Table          | Status            | Insertion time   |
| -------------: | :---------------: | :--------------: |
| `t_norm`       | Ordinary table    | 4.7 secs         |
| `t_temp`       | Temporary table   | 1.8 secs         |
| `t_unlogged`   | Unlogged table    | 1.7 secs         |
| ============== | ================= | ================ |


<br/>

As you can see, timing for temporary and unlogged tables is pretty much the same, and this is because both are not inserted into WAL records, and therefore there is no *crash-recovery*  machinery involved. This also means that writing transactions against temporary and unlogged tables is much faster against those tables. **Of course, the above is not an absolute measurement of `INSERT` times, but is reported here just to give you an idea of differences**.

<br/>
Since there is a temporary table, *you need to keep opened the session with the master node or you are going to loose all the data in such table!*


## Doing the physical replication

Start a physical replication. This is not a tutorial about how to do a physical replication, I will report the commands I've done on a separate machine in order to get the replica cluster on its way:


<br/>
<br/>
```shell
% pg_basebackup -X stream --create-slot --slot 'carmensita_physical_replication_slot' -R -r 100M -D /postgres/12/replica -l "Test unlogged tables" -P -d "dbname=backup user=backup host=miguel" -T /wal=/postgres/12
```
<br/>
<br/>

The original cluster is on a machine named `miguel`, while the replicated slot is placed on a machine named `carmensita`. These are the two machines I use always to do some experimental work.
<br/>
Please note also that I use a `backup` database and role to stream the information; as you can imagine you need to enable the replication connection on the `pg_hba.conf`:

<br/>
<br/>
```shell
% tail $PGDATA/pg_hba.conf

host    replication     backup  carmensita  trust
```
<br/>
<br/>
Once the replication has completed, you can fire up the standby node:


<br/>
<br/>
```shell
% /usr/pgsql-12/bin/pg_ctl -D /postgres/12/replica start 
in attesa che il server si avvii....
 LOG:  starting PostgreSQL 12.6 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 10.2.1 20201125 (Red Hat 10.2.1-9), 64-bit
 LOG:  listening on IPv4 address "0.0.0.0", port 5432
 LOG:  listening on IPv6 address "::", port 5432
 LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
 LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
 LOG:  redirecting log output to logging collector process
 HINT:  Future log output will appear in directory "log".
```
<br/>
<br/>


## Check the tables on the replication side

It is now time to check the replicated database on the replication host:


<br/>
<br/>
```sql
% psql -h carmensita -U luca rep_test

rep_test=> \d
                Lista delle relazioni
 Schema |       Nome        |   Tipo   | Proprietario 
--------|-------------------|----------|--------------
 public | t_norm            | tabella  | luca
 public | t_norm_pk_seq     | sequenza | luca
 public | t_unlogged        | tabella  | luca
 public | t_unlogged_pk_seq | sequenza | luca
(4 righe)

rep_test=> select count(*) from t_norm;
  count  
---------
 1000000
(1 riga)

rep_test=> select count(*) from t_unlogged;
ERROR:  cannot access temporary or unlogged relations during recovery
```
<br/>
<br/>

As you can see **the temporary table is missing**, even if still available on the other master connection.
There is no surprise here, a temporary table is *usable* only on a per-connection basis, and therefore will not be replicated.
<br/>
It is more interesting to see that the unlogged table `t_unlogged` and the related sequence have been replicated, but **they are there only as a placeholder**, and in fact it is not possible to act on the unlogged table.
<br/>
**Therefore unlogged tables are replicated in their structure but not in their data!**


## Switching from unlogged to logged

On the master node, it is now time to change the unlogged status of `t_unlogged` to *logged*, and this can be done quickly with the `ALTER TABLE` command.
Let's also check the status of the `relpersistence` flag on `pg_class` to see how it changed from `u` (*u*nlogged) to `p` (*p*ersistent):


<br/>
<br/>
```sql
rep_test=> \d
                Lista delle relazioni
 Schema |       Nome        |   Tipo   | Proprietario 
--------|-------------------|----------|--------------
 public | t_norm            | tabella  | luca
 public | t_norm_pk_seq     | sequenza | luca
 public | t_unlogged        | tabella  | luca
 public | t_unlogged_pk_seq | sequenza | luca
(4 righe)

rep_test=> select count(*) from t_norm;
  count  
---------
 1000000
(1 riga)

rep_test=> select count(*) from t_unlogged;
ERROR:  cannot access temporary or unlogged relations during recovery
```
<br/>
<br/>

The interesting part to note here is that changing the unlogged status to logged required `11 secs`, that is more than the the insertion time on an ordinary table. The idea here is that PostgreSQL has to *insert into the WALs all the records* from the table, as the `INSERT` of each row just happened.

<br/>
<br/>
```sql
rep_test=# alter table t_unlogged set logged;
ALTER TABLE
Time: 11485.505 ms (00:11.486)
```
<br/>
<br/>

and after that, on the replicated standby the table becomes *ordinary* too:


<br/>
<br/>
```sql
rep_test=> select count(*) from t_unlogged;
  count  
---------
 1000000
```
<br/>
<br/>


## Switching from logged to unlogged

What happens now if the `t_unlogged` returns unlogged again:


<br/>
<br/>
```sql
rep_test=# alter table t_unlogged set unlogged;
ALTER TABLE
Time: 5236.165 ms (00:05.236)
rep_test=# truncate t_unlogged;
TRUNCATE TABLE
Time: 21.498 ms
```
<br/>
<br/>

The interesting part to note here is that, again, there is a lot of time spent in the storage change.
<br/>
On the standby, the table become again not usable:

<br/>
<br/>
```sql
rep_test=> select count(*) from t_unlogged;
ERROR:  cannot access temporary or unlogged relations during recovery
rep_test=> select relpages, reltuples from pg_class where oid = 't_unlogged'::regclass;
 relpages | reltuples 
----------|-----------
        0 |         0

```
<br/>
<br/>

## Does the replica knows about the unlogged tables?

Of course it does, and in fact `pg_class` knows how many tuples and pages the table is using.
<br/>
However the table **is not consuming store space on the replication host**. In other words, the database on the replication side knows how much the table occupies on the master node, because the `pg_class` (and other catalogs) are replicated too. The table data is missing on disk.

<br/>
Let's see this on the master side:

<br/>
<br/>
```sql
rep_test=# select relpages, reltuples, 
                  pg_size_pretty( pg_relation_size( 't_unlogged') ), 
                  pg_relation_filepath( oid ) 
                  from pg_class where oid = 't_unlogged'::regclass;
 relpages | reltuples | pg_size_pretty | pg_relation_filepath 
----------|-----------|----------------|----------------------
    12738 |     2e+06 | 100 MB         | base/41441/41555

```
<br/>
<br/>

and on disk the size of the file is

<br/>
<br/>
```shell
% sudo du -h $PGDATA/base/41441/41555
100M    /postgres/12/data/base/41441/41555
```
<br/>
<br/>

What on the replicating host? The information is the same, but on the disk there is nothing:

<br/>
<br/>
```sql
rep_test=# select relpages, reltuples, 
                  pg_size_pretty( pg_relation_size( 't_unlogged') ), 
                  pg_relation_filepath( oid ) 
                  from pg_class where oid = 't_unlogged'::regclass;
                  
 relpages | reltuples | pg_size_pretty | pg_relation_filepath 
----------|-----------|----------------|----------------------
    12738 |     2e+06 | 0 bytes        | base/41441/41555
                 
```
<br/>
<br/>

and on disk, in fact, there is no room occupied by the table:


<br/>
<br/>
```shell
% sudo du -h /postgres/12/replica/base/41441/4155
0       /postgres/12/replica/base/41441/4155
```
<br/>
<br/>

## Unlogged but replicated ~ ordinary

An unlogged table that is replicated, looses the speed advantages of being unlogged.
<br/>
Why?
Because the system has to provide all the machinery to synchronize the table once it becomes logged.
If you "stop" the replication, removing the slots and other related stuff, the table gains speed.



# Conclusions

As expected, PostgreSQL replicates only *logged* tables and not *temporary* or *unlogged* ones. The latter are however present on the replicating side as *placeholders*, and once you turn them as logged they are fully shipped to the replicating part.
