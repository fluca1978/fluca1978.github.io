---
layout: post
title:  "How much data goes into the WALs? (part 2)" 
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
I did some more experiments with WALs.

# How much data goes into the WALs? (part 2**

In order to get a better idea about how WAL settings can change the situation within the WAL management, I decided to run a kind of automated test and store the results into a table, so that I can query them back later.
<br/>
The idea is the same of [my previous article](https://fluca1978.github.io/2021/07/13/PostgreSQLWalTraffic.html){:target="_blank"}: produce some workload, meausere the differences in the Log Sequence Numbers, and see how the size of WALs change depending on some setting. This is not an accurate research, it's just a quick and dirty experiment.

<br/>

At the end, I decided to share my numbers so that you can have a look at them and elaborate a bit more. For example, I'm no good at all at doing graphs (I know only the very minimum about `gnuplot`!).

### !!! WARNING !!!
**WARNING: this is not a guide on how to tune WAL settings!** 
*This is not even a real and comprhensive set of experiments*, it is just what I've played with to see how much traffic can be generated for certain amount of workloads.
<br/>
Your case and situation could be, and probably is, different from the very simple test I've done, and I do not pretend to be right about the small and obvious conclusions I come up at the end. In the case you see or know something that can help making more clear what I write in the following, please comment or contact me!


## Set up

First of all I decided to run an `INSERT` only workload, so that the size of the resulting table does not include any bloating and is therefore *comparable* to the effort about the WAL records.
<br/>
No other database activity was ongoing, so that the only generated WAL traffic was about my own workload.
<br/>
Each time the configuration was changed, the system was restarted, so that every workload started with the same (empty) clean situation and without any need to reason about ongoing checkpoints. Of course, checkpoints were happening as usual, but not at the beginning of the workload.
<br/>
<br/>
I used two tables to run the test:
- `wal_traffic` stores the results of each run;
- `wal_traffic_data` is used to store the data about every workload, that is tuples inserted in the database.
<br/>
The `wal_traffic_data` was dropped and re-created every time a new run was started, so to avoid data bloating
It is interesting to note that any workload setup activity is performed before the server is restarted, so that the only WAL traffic measured is as close as possible to the workload only.
<br/>
The `wal_traffic` table is defined as follows:

<br/>
<br/>
```sql
CREATE TABLE IF NOT EXISTS wal_traffic
  (
    pk int generated always as identity
    , workload text
    , lsn_start pg_lsn
    , lsn_end   pg_lsn
    , lsn_insert_start pg_lsn
    , lsn_insert_end   pg_lsn
    , run int          default 0
    , data_size bigint default 0
    , wal_size bigint generated always as ( lsn_end - lsn_start ) stored
    , wal_data_ratio numeric generated always as ( ( lsn_end - lsn_start )::real / data_size * 100 ) stored
    , wal_insert_data_ratio numeric generated always as ( ( lsn_insert_end - lsn_insert_start )::real / data_size * 100 ) stored
    , settings jsonb
    , workload_repetitions int default 1
    , ts_start timestamp default current_timestamp
    , ts_end   timestamp default current_timestamp

    , PRIMARY KEY ( pk )
  );
```
<br/>
<br/>
The `workload` field stores the text string about the executed query.
<br/>
The `lsn_xxx` fields store the location within the WAL, in particular:
- `lsn_start` and `lsn_end` store the result of `pg_current_wal_lsn()` function invoked at the begin and at the end of the workload;
- `lsn_insert_start` and `lsn_insert_end` store the result of `pg_current_wal_insert_lsn()` function invoked at the beginning and ending of the workload.
<br/>

I decided to store both the information to be able to examine differences in a more accurate way, however for this kind of experiment the differences between the values are pretty much useless.
<br/>
The `data_size` column contains the result of `pg_relation_size()`, that is a rough estimation of the volumen of data produced during the workload.
<br/>
The columns `wal_size`, `wal_data_ratio`, and `wal_insert_data_ratio`are generated, and contain repsectively the amount of generated WAL records and the ratio between the size of the *actual* data and that of the WAL records.
<br/>
Last, the `settings` column contains a `jsonb` representation of the settings used to run the test, like for example the value for `wal_level`, `wal_compression` and so on.

<br/>
<BR/>
There is also a view to quickly get results about the workload size:

<br/>
<br/>
```sql
CREATE OR REPLACE VIEW vw_wal_traffic
  AS
  select pg_size_pretty( data_size ) as data_size,
		 pg_size_pretty( wal_size ) as wal_size, wal_data_ratio::numeric( 7, 2 ) || ' %' as ratio,
		 wal_insert_data_ratio::numeric( 7, 2 ) || '%' as ins_ratio,
		 ts_end - ts_start as elapsed_time,
		 settings from wal_traffic;
```
<br/>
<br/>




## Details about the workloads

I've prepared two different workload, both based on `INSERT`s.
<br/>
The first workload does two transactions: the first one inserts a certain amount of tuples, while the second inserts a smaller amount of tuples. In particular, the first transaction inserts a  number of tuples specified by `$workload_scale`, while the second transaction inserts `1/5` of the same value.

<br/>
<br/>
```sql
BEGIN;
INSERT INTO $WORKLOAD_TABLE SELECT v, md5( v::text )::text || random()::text
FROM generate_series( 1, $workload_scale ) v;
COMMIT;

BEGIN;
INSERT INTO $WORKLOAD_TABLE 
SELECT v + v, t || ' - ' || t || random()::text
FROM $WORKLOAD_TABLE
WHERE v % 5 = 0;
COMMIT;
```

<br/>
<br/>
The `$workload_scale` variable assumes the values ranging from `100` to `10 million` growing by a factor of ten (e.g., `100`, `1000`, `10000` and so on).
<br/>
The second workload type is shorter, and does the following:


<br/>
<br/>
```sql
DO $$
DECLARE
  i int;
BEGIN
  FOR i IN 1 .. $workload_scale LOOP
    INSERT INTO $WORKLOAD_TABLE SELECT 1, md5( random()::text )::text;
  END LOOP;
END
$$;
```
<br/>
<br/>

Therefore performs the same number of tuple insertions as in the previous transaction, but it does by looping.
The final effect is that the first workload executes a single `INSERT` statetement, while the second workload executes several `INSERT` statements.
<br/>
<br/>
The usage of `random()` within the `INSERT` statements is to generate some more traffic on logical decoding.



## The Workload Workflow

In order to do the tests, I wrote an ugly shell script with the following workflow:
- truncate the `wal_traffic_data` table, so that its size on disk does not include previous experiments;
- execute a few `ALTER SYSTEM` to set some configuration on WAL related parameters (`wal_level`, `full_page_writes`, `wal_compression` and so on);
- restart the PostgreSQL system, so to ensure every test has a clean and clear situation;
- get the current WAL position (`pg_current_wal_lsn()` and `pg_current_wal_insert_lsn()`);
- execute the workload with the right *scale*;
- get the current WAL position (`pg_current_wal_lsn()` and `pg_current_wal_insert_lsn()`);
- insert the result tuple with WAL differences into `wal_traffic`;
- loop with a different scaling factor.


# The Results

It is now time to have a look at the test results.

Let's consider a few results:

<br/>
<br/>
```sql
testdb=> select * from vw_wal_traffic where settings->>'wal_level' = 'minimal' and settings->>'wal_compression' = 'on';
-[ RECORD 1 ]+-----------------------------------------------------------------------------------------------------
data_size    | 1205 MB
wal_size     | 2148 MB
ratio        | 178.27 %
ins_ratio    | 178.27%
elapsed_time | 00:02:33.282366
settings     | {"wal_level": "minimal", "wal_log_hints": "off", "wal_compression": "on", "full_page_writes": "off"}
-[ RECORD 2 ]+-----------------------------------------------------------------------------------------------------
data_size    | 1205 MB
wal_size     | 2148 MB
ratio        | 178.27 %
ins_ratio    | 178.27%
elapsed_time | 00:02:34.882126
settings     | {"wal_level": "minimal", "wal_log_hints": "on", "wal_compression": "on", "full_page_writes": "on"}

```
<br/>
<br/>
As you can see, for `1,2 GB` of data the system has produced roughly `2,1 GB` of WAL records. And the situation is even worst when there is no `wal_compression` (as you could expect):

<br/>
<br/>
```sql
-[ RECORD 8 ]+------------------------------------------------------------------------------------------------------
data_size    | 1205 MB
wal_size     | 2402 MB
ratio        | 199.34 %
ins_ratio    | 199.34%
elapsed_time | 00:02:30.725138
settings     | {"wal_level": "minimal", "wal_log_hints": "off", "wal_compression": "off", "full_page_writes": "on"}

```
<br/>
<br/>
this time, for the same amount of data, the WAL size is almost double that of the real data.
<br/>
Changing the setting of `wal_level` to `logical` or `replicat` does not change very much the situation, 

<br/>
It is possible to get the best ratio between the WAL produced and the data stored:


<br/>
<br/>
```sql
testdb=> select * from vw_wal_traffic v where ratio = ( select min( ratio ) from vw_wal_traffic where settings->>'wal_level' = v.settings->>'wal_level' ) and v.settings->>'wal_level' IN ( 'minimal', 'replica', 'logical' );
-[ RECORD 1 ]+-----------------------------------------------------------------------------------------------------
data_size    | 16 kB
wal_size     | 16 kB
ratio        | 101.95 %
ins_ratio    | 101.95%
elapsed_time | 00:00:00.133674
settings     | {"wal_level": "logical", "wal_log_hints": "on", "wal_compression": "on", "full_page_writes": "off"}
-[ RECORD 2 ]+-----------------------------------------------------------------------------------------------------
data_size    | 16 kB
wal_size     | 16 kB
ratio        | 101.56 %
ins_ratio    | 101.56%
elapsed_time | 00:00:00.120578
settings     | {"wal_level": "replica", "wal_log_hints": "on", "wal_compression": "on", "full_page_writes": "on"}
-[ RECORD 3 ]+-----------------------------------------------------------------------------------------------------
data_size    | 16 kB
wal_size     | 18 kB
ratio        | 111.13 %
ins_ratio    | 100.34%
elapsed_time | 00:00:00.427126
settings     | {"wal_level": "minimal", "wal_log_hints": "off", "wal_compression": "on", "full_page_writes": "off"}

```
<br/>
<br/>
and on the other side, the worst ratio:


<br/>
<br/>
```sql
testdb=> select * from vw_wal_traffic v where ratio = ( select max( ratio ) from vw_wal_traffic where settings->>'wal_level' = v.settings->>'wal_level' ) and v.settings->>'wal_level' IN ( 'minimal', 'replica', 'logical' );  
-[ RECORD 1 ]+-----------------------------------------------------------------------------------------------------
data_size    | 8192 bytes
wal_size     | 23 kB
ratio        | 289.16 %
ins_ratio    | 190.72%
elapsed_time | 00:00:00.266881
settings     | {"wal_level": "minimal", "wal_log_hints": "off", "wal_compression": "off", "full_page_writes": "on"}
-[ RECORD 2 ]+-----------------------------------------------------------------------------------------------------
data_size    | 8192 bytes
wal_size     | 23 kB
ratio        | 289.16 %
ins_ratio    | 190.72%
elapsed_time | 00:00:00.112946
settings     | {"wal_level": "minimal", "wal_log_hints": "off", "wal_compression": "off", "full_page_writes": "on"}
-[ RECORD 3 ]+-----------------------------------------------------------------------------------------------------
data_size    | 8192 bytes
wal_size     | 23 kB
ratio        | 284.47 %
ins_ratio    | 190.63%
elapsed_time | 00:00:00.076021
settings     | {"wal_level": "logical", "wal_log_hints": "off", "wal_compression": "off", "full_page_writes": "on"}
-[ RECORD 4 ]+-----------------------------------------------------------------------------------------------------
data_size    | 8192 bytes
wal_size     | 23 kB
ratio        | 289.65 %
ins_ratio    | 190.53%
elapsed_time | 00:00:00.113793
settings     | {"wal_level": "replica", "wal_log_hints": "off", "wal_compression": "off", "full_page_writes": "on"}


```
<br/>
<br/>
From the above, it is clear that the worst cases are those with `wal_compression` disabled, while the best cases are those with compression enabled.


<br/>


## Download the Results

The [results are available by means of a CSV file](https://gitlab.com/fluca1978/fluca1978-pg-utils/-/blob/master/examples/wal_traffic/wal_traffic.csv){:target="_blank"}, so you can load and inspect them yourself.
In order to load the files, create a table `wal_traffic_results` as follows:

<br/>
<br/>
```sql
testdb=> create table wal_traffic_results ( 
   run int, workload text, wal_size bigint, 
   data_size bigint, 
   wal_data_ratio numeric( 5,2), 
   wall_clock time, wal_level text, 
   wal_log_hints text, 
   wal_compression text, 
   full_page_writes text );
```
<br/>
<br/>
and then load the CSV file with a command like the following one:


<br/>
<br/>
```sql
testdb=> \copy wal_traffic_results from wal_traffic.csv with ( format csv, header  );
```
<br/>
<br/>

Please note that I've split the `jsonb` field into a set of columns with a query like the following one, that produced the CSV file:

<br/>
<br/>
```sql 
% psql -A --csv -h miguel 
-c 'select run, workload, wal_size, data_size, wal_data_ratio,  ts_end - ts_start as wall_clock, x.* from wal_traffic
cross join lateral jsonb_to_record( settings ) as x( wal_level text, wal_log_hints text, wal_compression text, full_page_writes text );' 
testdb >! wal_traffic.csv
```
<br/>
<br/>


## More Results

From the de-`jsonb` representation of the results, it is easier to get a glance at the WAL ratio by workload type

<br/>
<br/>
```sql
testdb=> select workload, min( wal_data_ratio ), max( wal_data_ratio ), max( wal_data_ratio ) - min( wal_data_ratio ) as diff
from wal_traffic_results 
group by workload order by 4 asc;
-[ RECORD 1 ]-------------------------------------------------------------------------------
workload | 'BEGIN;                                                                          +
         | INSERT INTO wal_traffic_workload SELECT v, md5( v::text )::text || random()::text+
         | FROM generate_series( 1, 1000000 ) v;                                            +
         | COMMIT;                                                                          +
         |                                                                                  +
         | BEGIN;                                                                           +
         | INSERT INTO wal_traffic_workload                                                 +
         | SELECT v + v, t || '' - '' || t || random()::text                                +
         | FROM wal_traffic_workload                                                        +
         | WHERE v % 5 = 0;                                                                 +
         | COMMIT;'
min      | 125.55
max      | 125.60
diff     | 0.05
-[ RECORD 2 ]-------------------------------------------------------------------------------
workload | 'DO $wl$ DECLARE                                                                 +
         |   i int;                                                                         +
         | BEGIN                                                                            +
         |   FOR i IN 1 .. 1000000 LOOP                                                     +
         |     INSERT INTO wal_traffic_workload SELECT 1, md5( random()::text )::text;      +
         |   END LOOP;                                                                      +
         | END $wl$;'
min      | 125.76
max      | 125.82
diff     | 0.06
-[ RECORD 3 ]-------------------------------------------------------------------------------
workload | 'DO $wl$ DECLARE                                                                 +
         |   i int;                                                                         +
         | BEGIN                                                                            +
         |   FOR i IN 1 .. 10000000 LOOP                                                    +
         |     INSERT INTO wal_traffic_workload SELECT 1, md5( random()::text )::text;      +
         |   END LOOP;                                                                      +
         | END $wl$;'
min      | 125.76
max      | 125.92
diff     | 0.16
-[ RECORD 4 ]-------------------------------------------------------------------------------
workload | 'BEGIN;                                                                          +
         | INSERT INTO wal_traffic_workload SELECT v, md5( v::text )::text || random()::text+
         | FROM generate_series( 1, 100000 ) v;                                             +
         | COMMIT;                                                                          +
         |                                                                                  +
         | BEGIN;                                                                           +
         | INSERT INTO wal_traffic_workload                                                 +
         | SELECT v + v, t || '' - '' || t || random()::text                                +
         | FROM wal_traffic_workload                                                        +
         | WHERE v % 5 = 0;                                                                 +
         | COMMIT;'
min      | 125.49
max      | 125.73
diff     | 0.24
-[ RECORD 5 ]-------------------------------------------------------------------------------
workload | 'DO $wl$ DECLARE                                                                 +
         |   i int;                                                                         +
         | BEGIN                                                                            +
         |   FOR i IN 1 .. 100000 LOOP                                                      +
         |     INSERT INTO wal_traffic_workload SELECT 1, md5( random()::text )::text;      +
         |   END LOOP;                                                                      +
         | END $wl$;'
min      | 125.72
max      | 125.97
diff     | 0.25
-[ RECORD 6 ]-------------------------------------------------------------------------------
workload | 'BEGIN;                                                                          +
         | INSERT INTO wal_traffic_workload SELECT v, md5( v::text )::text || random()::text+
         | FROM generate_series( 1, 10000 ) v;                                              +
         | COMMIT;                                                                          +
         |                                                                                  +
         | BEGIN;                                                                           +
         | INSERT INTO wal_traffic_workload                                                 +
         | SELECT v + v, t || '' - '' || t || random()::text                                +
         | FROM wal_traffic_workload                                                        +
         | WHERE v % 5 = 0;                                                                 +
         | COMMIT;'
min      | 124.99
max      | 126.55
diff     | 1.56
-[ RECORD 7 ]-------------------------------------------------------------------------------
workload | 'DO $wl$ DECLARE                                                                 +
         |   i int;                                                                         +
         | BEGIN                                                                            +
         |   FOR i IN 1 .. 10000 LOOP                                                       +
         |     INSERT INTO wal_traffic_workload SELECT 1, md5( random()::text )::text;      +
         |   END LOOP;                                                                      +
         | END $wl$;'
min      | 125.14
max      | 127.47
diff     | 2.33
-[ RECORD 8 ]-------------------------------------------------------------------------------
workload | 'BEGIN;                                                                          +
         | INSERT INTO wal_traffic_workload SELECT v, md5( v::text )::text || random()::text+
         | FROM generate_series( 1, 10000000 ) v;                                           +
         | COMMIT;                                                                          +
         |                                                                                  +
         | BEGIN;                                                                           +
         | INSERT INTO wal_traffic_workload                                                 +
         | SELECT v + v, t || '' - '' || t || random()::text                                +
         | FROM wal_traffic_workload                                                        +
         | WHERE v % 5 = 0;                                                                 +
         | COMMIT;'
min      | 178.27
max      | 199.46
diff     | 21.19
-[ RECORD 9 ]-------------------------------------------------------------------------------
workload | 'BEGIN;                                                                          +
         | INSERT INTO wal_traffic_workload SELECT v, md5( v::text )::text || random()::text+
         | FROM generate_series( 1, 1000 ) v;                                               +
         | COMMIT;                                                                          +
         |                                                                                  +
         | BEGIN;                                                                           +
         | INSERT INTO wal_traffic_workload                                                 +
         | SELECT v + v, t || '' - '' || t || random()::text                                +
         | FROM wal_traffic_workload                                                        +
         | WHERE v % 5 = 0;                                                                 +
         | COMMIT;'
min      | 121.58
max      | 152.01
diff     | 30.43
-[ RECORD 10 ]------------------------------------------------------------------------------
workload | 'DO $wl$ DECLARE                                                                 +
         |   i int;                                                                         +
         | BEGIN                                                                            +
         |   FOR i IN 1 .. 1000 LOOP                                                        +
         |     INSERT INTO wal_traffic_workload SELECT 1, md5( random()::text )::text;      +
         |   END LOOP;                                                                      +
         | END $wl$;'
min      | 118.45
max      | 167.37
diff     | 48.92
-[ RECORD 11 ]------------------------------------------------------------------------------
workload | 'BEGIN;                                                                          +
         | INSERT INTO wal_traffic_workload SELECT v, md5( v::text )::text || random()::text+
         | FROM generate_series( 1, 100 ) v;                                                +
         | COMMIT;                                                                          +
         |                                                                                  +
         | BEGIN;                                                                           +
         | INSERT INTO wal_traffic_workload                                                 +
         | SELECT v + v, t || '' - '' || t || random()::text                                +
         | FROM wal_traffic_workload                                                        +
         | WHERE v % 5 = 0;                                                                 +
         | COMMIT;'
min      | 101.56
max      | 247.46
diff     | 145.90
-[ RECORD 12 ]------------------------------------------------------------------------------
workload | 'DO $wl$ DECLARE                                                                 +
         |   i int;                                                                         +
         | BEGIN                                                                            +
         |   FOR i IN 1 .. 100 LOOP                                                         +
         |     INSERT INTO wal_traffic_workload SELECT 1, md5( random()::text )::text;      +
         |   END LOOP;                                                                      +
         | END $wl$;'
min      | 124.02
max      | 289.65
diff     | 165.63
                            
```
<br/>
<br/>
There are  certain workload (by type and size) that do not produce any sensible variation in the WAL produced, while for example the last workload for a small amount of tuples produces a very wide range of WAL record writes.
<br/>
We could also query to search for a *trend* in the ratio:

<br/>
<br/>
```sql
testdb=> select wal_data_ratio, wal_level, wal_log_hints, wal_compression, full_page_writes from wal_traffic_results where workload like '%FOR i IN 1 .. 100 LOOP%' order by 1 desc;
-[ RECORD 1 ]----|--------
wal_data_ratio   | 289.65
wal_level        | replica
wal_log_hints    | off
wal_compression  | off
full_page_writes | on

...
-[ RECORD 6 ]----|--------
wal_data_ratio   | 256.35
wal_level        | logical
wal_log_hints    | on
wal_compression  | off
full_page_writes | on
...
-[ RECORD 19 ]---|--------
wal_data_ratio   | 150.78
wal_level        | replica
wal_log_hints    | on
wal_compression  | on
full_page_writes | off
...
-[ RECORD 36 ]---|--------
wal_data_ratio   | 124.02
wal_level        | replica
wal_log_hints    | on
wal_compression  | on
full_page_writes | on

```
<br/>
<br/>
The above confirms how much `wal_compression` is going to reduce the WAL traffic.
<br/>
And again, the `wal_level` is not going to influence the WAL size too much:


<br/>
<br/>
```sql
testdb=> select min( wal_data_ratio ), max( wal_data_ratio ), wal_level 
         from wal_traffic_results 
         where workload like '%FOR i IN 1 .. 100 LOOP%'  
         group by wal_level order by 1 desc, 2 desc;
-[ RECORD 1 ]------
min       | 146.00
max       | 289.16
wal_level | minimal
-[ RECORD 2 ]------
min       | 124.12
max       | 284.47
wal_level | logical
-[ RECORD 3 ]------
min       | 124.02
max       | 289.65
wal_level | replica

```
<br/>
<br/>

# Conclusions

Even a small amount of *real data* can produce quite a lot amount of WAL records, and this is good because within those records there are all the information PostgreSQL needs to keep our data at safe, that after all its our final goal.
<br/>
WAL related settings can, of course, influence the amount of generated data and the idea behind this article is not to provide an exhaustive guide to tune WALs, rather to show how you can measure your WAL traffic depending on the workload you are facing.
<br/>
This should then help you to decide the right way to tune your WALs.
<br>
In the case you find something wrong in the approach described above, or want to integrate or share your experience, please comment on contact me.
