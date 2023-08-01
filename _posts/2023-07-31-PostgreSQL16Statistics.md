---
layout: post
title:  "PostgreSQL 16 introduces a few new statistic fields for tables and indexes"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
An addition to the pg_stat_xxx_tables and pg_stat_xxx_indexes that can help a lot in finding out seldomly used stuff.

# PostgreSQL 16 introduces a few new statistic fields for tables and indexes

PostgreSQL 16 adds two important timestamp fields to the statistics about tables and indexes, most notably `pg_stat_all_tables` and `pg_stat_all_indexes`. Clearly, such fields are also *inherited* in user and system catalogs, like for instance `pg_stat_user_tables` and `pg_stat_user_indexes`.
These two fields contain the last time a sequential scan against a table or an index (i.e., the index was used to extract data, and hence read) happened. As for all things statistics in PostgreSQL, the information is not in real time, rather it is defined at a transaction boundary.
<br/>
<br/>
Before these two fields were added, the statistics catalog provided only quantitative information, clearly less accurate, because it required the database administrator to *guess* how the system was behaving, for example understanding if an index was unused. Clearly, a table with a huge quantitative value of sequential scans is a good candidate for a few indexes to be created, while on the other hand an index with a very low usage counter is a good candidate for removal. With PostgreSQL 16 is now possible to better decide what to do in the above cases, understanding **how far in the past** a particular event happened, and hence better understand how acting on such table or index will affect the database.

<br/>

In this article, I give a very short presentation of what it is like to query the new fields in the statistics catalogs. The tests have been done on PostgreSQL 16 beta-2.



## Creating a simple workbench

Let's create a very simple table and populate it:

<br/>
<br/>
```sql
testdb=> CREATE TABLE t( pk int generated always as identity
         , t text
        , primary key ( pk ) );
CREATE TABLE

testdb=> insert into t( t )
         select 'Test tuple #' || v
        from generate_series(1, 10000000 ) v;
INSERT 0 10000000

```
<br/>
<br/>

and add an index to the table

<br/>
<br/>
```sql
testdb=> CREATE INDEX idx_t_even ON t( pk ) WHERE pk % 2 = 0;
CREATE INDEX
```
<br/>
<br/>

The above bring up a table and index with the following sizes:

<br/>
<br/>
```sql
testdb=> SELECT relname, pg_size_pretty( pg_relation_size( oid ) )
FROM pg_class
WHERE relname IN ( 't', 'idx_t_even', 't_pkey' );
  relname   | pg_size_pretty
------------+----------------
 t          | 498 MB
 t_pkey     | 214 MB
 idx_t_even | 107 MB
(3 rows)

```
<br/>
<br/>


## Let's have a look at the statistics

In the `pg_stat_xxx_tables` and `pg_stat_xxx_indexes` there are now two new fields named `last_seq_scan` and `last_idx_scan` respectively. These fields are timestamps and contain the timestamp of the last time a sequential scan or an index scan has been performed.

For example:

<br/>
<br/>
```sql
testdb=> SELECT relname, seq_scan, now() - last_seq_scan as seq_scan_age, idx_scan
         FROM pg_stat_user_tables WHERE relname = 't';
-[ RECORD 1 ]+----------------
relname      | t
seq_scan     | 3
seq_scan_age | 00:04:57.627383
idx_scan     | 0

```
<br/>
<br/>

that gives the idea that the table has been read sequentially three times, last of which near five minutes ago.
And in fact, if we perform another query on the table, the statistics gets update:

<br/>
<br/>
```sql
testdb=> SELECT count(*) FROM t;
-[ RECORD 1 ]---
count | 10000000

testdb=> SELECT relname, seq_scan, now() - last_seq_scan as seq_scan_age, idx_scan FROM pg_stat_user_tables WHERE relname = 't';
-[ RECORD 1 ]+----------------
relname      | t
seq_scan     | 6
seq_scan_age | 00:00:01.508468
idx_scan     | 0

```
<br/>
<br/>


What about the index? Well, the `pg_stat_user_indexes` shows information about the indexes and, in this case, the `last_idx_scan` is the added field:

<br/>
<br/>
```sql
testdb=> SELECT relname, indexrelname, idx_scan, now() - last_idx_scan FROM pg_stat_user_indexes WHERE relname = 't';
-[ RECORD 1 ]+-----------
relname      | t
indexrelname | t_pkey
idx_scan     | 0
?column?     |
-[ RECORD 2 ]+-----------
relname      | t
indexrelname | idx_t_even
idx_scan     | 0
?column?     |

```
<br/>
<br/>

Even in this case, when the index is used the `last_idx_scan` field is updated accordingly:

<br/>
<br/>
```sql
testdb=> SELECT relname, indexrelname, idx_scan, now() - last_idx_scan FROM pg_stat_user_indexes WHERE relname = 't';
-[ RECORD 1 ]+----------------
relname      | t
indexrelname | t_pkey
idx_scan     | 0
?column?     |
-[ RECORD 2 ]+----------------
relname      | t
indexrelname | idx_t_even
idx_scan     | 1
?column?     | 00:00:01.885197

```
<br/>
<br/>


# Conclusions

Before PostgreSQL 16, the `pg_stat_xxx_tables` and `pg_stat_xxx_indexes` provided only quantitative information about the number of sequential scans and index usage, now it is also possible to have an idea on when such event last happened. This is important because it can reveal quickly how your indexes are performing without requiring you to reset the statistics and start monitoring them from scratch.
