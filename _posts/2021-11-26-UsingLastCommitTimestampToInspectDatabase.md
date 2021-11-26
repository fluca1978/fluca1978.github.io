---
layout: post
title:  "Monitoring Schema Changes via Last Commit Timestamp"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
An ugly way to introspect database changes.

# Monitoring Schema Changes via Last Commit Timestamp

A few days ago, a colleague of mine shown to me that a [commercial database keeps track of *last DDL change timestamp* against database objects](https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2005.htm#i1583352){:target="_blank"}. 
<br/>
I began to mumble... is that possible in PostgreSQL? Of course it is, but what is the smartest way to achieve it?
<br/>
I asked on the [mailing list](https://www.postgresql.org/message-id/CAKoxK%2B5uz67DxYcF%3D6ese6imi7ovHoqwjB%2BYE4X46sNv_9KN0w%40mail.gmail.com){:target="_blank"}, because the first idea that came into my mind was to use *commit timestamps*.
<br/>
Clearly, it is possible to implement something that can do the job using *event triggers*, that in short are triggers not attached to table tuples rather to database event like DDL commands. Great! And in fact, a very good explaination [can be found here](https://www.depesz.com/2012/07/29/waiting-for-9-3-event-triggers/){:target="_blank"}.
<br/>
In this article, I present my first idea about using commit timestamps.
<br/>
The system used for the test is PostgreSQL 13.4 running on Fedora Linux, with only myself connected to it (this simplifies following transactions). The idea is, in any case, general and easy enough to be used on busy systems.


# Introduction to `pg_last_committed_xact()`

The special function [`pg_last_committed_xact()`](https://www.postgresql.org/docs/current/functions-info.html){:target="_blank"} allows the database administrator (or an user) to get information about which transaction has committed last.
<br/>
Let's see this in action:


<br/>
<br/>
```shell
% psql -U luca -h miguel -c 'select pg_last_committed_xact();'   testdb
ERROR:  could not get commit timestamp data
HINT:  Make sure the configuration parameter "track_commit_timestamp" is set.
```
<br/>
<br/>

**First of all** in order to get information about the committed transaction timestamps, there must be the option `track_commit_timestamp` configured.
<br/>
Turning on and off the parameter will not provide historic data, that is even if you *had* the parameter on and then you turned off, you will not be able to access collected data.
<br/>
Let's turn on the parameter and see how it works. **The `track_commit_timestamp` is a parameter with the `postmaster` context, and therefore requires a server restart!**


<br/>
<br/>
```shell
% psql -U postgres -h miguel \
       -c 'ALTER SYSTEM SET track_commit_timestamp to "on"; ' \
       testdb
ALTER SYSTEM
% ssh luca@miguel 'sudo systemctl restart postgresql-13'
```
<br/>
<br/>

In the above I restarted a remote system via `ssh`, of course you are free to configure the parameter and restart the cluster with your preferred (or available) method.
<br/>
It is now time to see which information we can get with `track_commit_timestamp` turned on. 

<br/>
<br/>
```sql
testdb=> SELECT txid_current();
-[ RECORD 1 ]+-------------
txid_current | 380316302458

testdb=> SELECT *  FROM  txid_status( 380316302457 ), 
                         pg_last_committed_xact();
-[ RECORD 1 ]------------------------------
txid_status | committed
xid         | 2359180410
timestamp   | 2021-11-20 04:28:50.223275-05

```
<br/>
<br/>

Let's dissect the above example:
- `txid_current()` simulates a new transaction in one row, because the function gets a new `xid` (transaction identifier) even if not used for effective work;
- `txid_status()` accepts a xid identifier and returns a string with the status of the transaction, and as shown, the *fake* transaction `380316302458` results in status `committed`;
- `pg_last_committed_xact()` now is able to report both the `xid` and the `timestamp` at which the last transaction has committed, that is the transaction `380316302458` committed at `2021-11-20 04:28:50.223275-05`.

<br/>
Wait a minute: `pg_last_committed_xact()` states that the last committed transaction is `2359180410`, not `380316302458`. What is happening?
<br/>
**Wrap-around is on its way!**
<br/>
The above system has done a so called *xid wraparound*, that is normal situation in a long running PostgreSQL instance. What this means, is that `txid_current()` is resturning a *bumped* value that is, somehow, an absolute value. However, PostgreSQL "reasons" in terms of values modulo 2^32, therefore we must take into account this possible difference.
<br/>
The above example therefore becomes:

<br/>
<br/>
```sql
testdb=> SELECT txid_current() as xid_absolute, 
                mod( txid_current(), pow( 2, 32 )::bigint )  as xid;
-[ RECORD 1 ]+-------------
xid_absolute | 380316302460
xid          | 2359180412

testdb=> SELECT *  FROM  
              txid_status( 380316302460 ) as xid_abs_status, 
              txid_status( 2359180412 ) as xid_status,  
              pg_last_committed_xact();
-[ RECORD 1 ]--+------------------------------
xid_abs_status | committed
xid_status     | 
xid            | 2359180412
timestamp      | 2021-11-20 04:34:54.531106-05

```
<br/>
<br/>

The above demonstrates that transactions `380316302460` and `2359180412` are the same, according to PostgreSQL. However, `txid_status()` requires an "absolute" xid number (note how the short transaction number does not report any status), while `pg_last_committed_xact()` reasons in terms of "running" numbers, i.e., the modulo ones.
<br/>
There is another interesting function to keep in mind: `pg_xact_commit_timestamp()` that, given a transaction identifier, returns the known commit timestamp:

<br/>
<br/>
```sql
testdb=> SELECT * FROM 
                pg_xact_commit_timestamp( 2359180412::text::xid ), 
                pg_last_committed_xact();
-[ RECORD 1 ]------------+------------------------------
pg_xact_commit_timestamp | 2021-11-20 04:34:54.531106-05
xid                      | 2359180412
timestamp                | 2021-11-20 04:34:54.531106-05

```
<br/>
<br/>

As you can see, the timestamp for the same transaction is always the same. **Note that a `bigint` requires a conversion to `text` before being translated into a `xid`**.

# Tracking DDL Commands

Every table in PostgreSQL has two hidden fields that track the *transaction ranges*: `xmin` indicates the transaction that created a tuple, while `xmax` indicates the transaction that invalidated the tuple. This is used in the MVCC (Multi Version Concurrency Control) machinery that I'm not going to discuss here, so trust that everything works just fine.
<br/>
The keypoint here is: **every table has fields that track the transaction that generated the tuple**. This applies also to system catalogs, and in particular (with regard to this article) to `pg_class`.
<br/>
Having stated that, and knowing that every time a DDL command applies, something is changed in the system catalogs, it is therefore possible to track **when changes did happen** on a particular database object or table.
<br/>
Let's see this in action:

<br/>
<br/>
```sql
testdb=> BEGIN;
BEGIN
testdb=> SELECT txid_current() as xid_absolute, 
                mod( txid_current(), pow( 2, 32 )::bigint ) as xid, 
                current_timestamp;
-[ RECORD 1 ]-----+------------------------------
xid_absolute      | 380316302463
xid               | 2359180415
current_timestamp | 2021-11-20 05:11:56.343542-05

testdb=> CREATE TABLE ddl_test( 
           pk int generated always as identity, 
           t text );
CREATE TABLE
testdb=> COMMIT;
COMMIT

```
<br/>
<br/>

At timestamp `2021-11-20 05:11:56` the table `ddl_test` has been created. Since every DDL command in PostgreSQL is transactional, it is possible to track the transaction that committed such DDL (in the above example, `380316302463` alis `2359180415`).
<br/>
Let's query `pg_class` to get information about last DDL commands on `ddl_test` table:

<br/>
<br/>
```sql
testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , pg_xact_commit_timestamp( xmin ) as modified_at
                , relname as table
       FROM pg_class
       WHERE relkind = 'r'
       AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+------------------------------
transaction_before        | 1
it_was_transaction_number | 2359180415
modified_at               | 2021-11-20 05:12:21.359126-05
table                     | ddl_test

```
<br/>
<br/>

The above queries tells us that `1` transaction ago the transaction number `2359180415` modified the structure of `ddl_test` at timestamp `2021-11-20 05:12:21.359126-05**.
<br/>
**Everything seems fine except for the timestamp**: the transaction timestamp is not really the same as reported by `pg_xact_commit_timestamp()`. The reason for this is that *the moment a transaction commits is not the same as the transaction is consolidated*, therefore there could some offset and lag. However, checking deeper we can see that data is coherent:


<br/>
<br/>
```sql
testdb=> SELECT * FROM pg_last_committed_xact();
-[ RECORD 1 ]----------------------------
xid       | 2359180415
timestamp | 2021-11-20 05:12:21.359126-05
```
<br/>
<br/>

So this is a first *ugly* but pretty much unexpensive way to track changes to the table.

<BR/>
<BR/>
Let's now add a column to the table, so to see if this machinery can work:

<br/>
<br/>
```sql
testdb=> BEGIN;
BEGIN
testdb=> SELECT txid_current() as xid_absolute, mod( txid_current(), pow( 2, 32 )::bigint ) as xid, current_timestamp;
-[ RECORD 1 ]-----+------------------------------
xid_absolute      | 380316302464
xid               | 2359180416
current_timestamp | 2021-11-20 05:21:03.089031-05

testdb=> ALTER TABLE ddl_test ADD COLUMN tt text;
ALTER TABLE
testdb=> COMMIT;
COMMIT


testdb=> SELECT * FROM pg_last_committed_xact();
-[ RECORD 1 ]----------------------------
xid       | 2359180416
timestamp | 2021-11-20 05:21:32.376468-05
```

<br/>
<br/>

Transaction `2359180416` at timestamp `2021-11-20 05:21:32.376468-05` committed the `ALTER TABLE`. Let's run again our query against `pg_class`:


<br/>
<br/>
```sql
testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , pg_xact_commit_timestamp( xmin ) as modified_at
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+------------------------------
transaction_before        | 1
it_was_transaction_number | 2359180416
modified_at               | 2021-11-20 05:21:32.376468-05
table                     | ddl_test

```
<br/>
<br/>

Therefore we now know when the table was last *touched* by a DDL command.


## Going Deeper: Introspection Against Columns

From the above we now know *when* a change happened to our table, but we don't know which attribute has been changed. It is possible to *push* the same logic against other parts of the system catalog, for example `pg_attribute` that handles information about single table columns.
<br/>
Here it the example applied to our demo table:

<br/>
<br/>
```sql
testdb=> SELECT xmin, attname, age( xmin ), pg_xact_commit_timestamp( xmin ) 
FROM pg_attribute
WHERE attrelid = 'ddl_test'::regclass;
    xmin    | attname  | age |   pg_xact_commit_timestamp    
------------+----------+-----+-------------------------------
 2359180415 | tableoid |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | cmax     |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | xmax     |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | cmin     |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | xmin     |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | ctid     |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | pk       |   2 | 2021-11-20 05:12:21.359126-05
 2359180415 | t        |   2 | 2021-11-20 05:12:21.359126-05
 2359180416 | tt       |   1 | 2021-11-20 05:21:32.376468-05

```
<br/>
<br/>

All the columns except `tt` have been created by the very same transaction at the very same timestamp, while `tt` has been touched from another transation 11 minutes after.
<br/>
The above is not very useful, so it is possible to improve sligthly the query into the following one:

<br/>
<br/>
```sql
testdb=> SELECT array_agg( attname ) as columns,  
                current_timestamp - pg_xact_commit_timestamp( xmin ) as when 
         FROM pg_attribute
         WHERE attrelid = 'ddl_test'::regclass
         GROUP BY pg_xact_commit_timestamp( xmin );
                 columns                  |      when       
------------------------------------------+-----------------
 {tableoid,cmax,xmax,cmin,xmin,ctid,pk,t} | 00:19:38.202794
 {tt}                                     | 00:10:27.185452

```
<br/>
<br/>

That reports all the column "touched" at the very same time and how many time has elapsed from the last change. For example, the column `tt` has been changed 10 minutes ago, while the other columns 19 minutes ago.
<br/>
Let's do more changes to our table and see what happen; please note that everything is executed in autocommit mode:

<br/>
<br/>
```sql
testdb=> ALTER TABLE ddl_test ADD COLUMN ttt text;
ALTER TABLE

testdb=> ALTER TABLE ddl_test 
           ALTER COLUMN tt SET DEFAULT 'FizzBuzz';
ALTER TABLE


testdb=> ALTER TABLE ddl_test DROP COLUMN t;
ALTER TABLE

testdb=> SELECT * FROM pg_last_committed_xact();
    xid     |          timestamp           
------------+------------------------------
 2359180419 | 2021-11-20 05:36:48.54285-05
```
<br/>
<br/>

If we inspect again `pg_attribute` we have:

<br/>
<br/>
```sql
testdb=> SELECT array_agg( attname ) as columns,  
                current_timestamp - pg_xact_commit_timestamp( xmin ) as time_ago, 
                pg_xact_commit_timestamp( xmin ) as when       
         FROM pg_attribute
         WHERE attrelid = 'ddl_test'::regclass
         GROUP BY pg_xact_commit_timestamp( xmin );
                columns                 |    time_ago     |             when              
----------------------------------------+-----------------+-------------------------------
 {tableoid,cmax,xmax,cmin,xmin,ctid,pk} | 00:26:25.685984 | 2021-11-20 05:12:21.359126-05
 {ttt}                                  | 00:04:08.244367 | 2021-11-20 05:34:38.800743-05
 {tt}                                   | 00:02:31.791574 | 2021-11-20 05:36:15.253536-05
 {........pg.dropped.2........}         | 00:01:58.50226  | 2021-11-20 05:36:48.54285-05


testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , pg_xact_commit_timestamp( xmin ) as modified_at
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+------------------------------
transaction_before        | 3
it_was_transaction_number | 2359180417
modified_at               | 2021-11-20 05:34:38.800743-05
table                     | ddl_test


```
<br/>
<br/>

There are some interesting things in the above output.
First of all, `pg_class` reports only the changes related to *new* attributes, not the dropped ones or the internally changed. On the other hand, `pg_attribute` reports information about every single attribute, including those changed in a "minor" mode (the `SET DEFAULT` for instance).
<br/>
Please note how the dropped column (namely `t`) is no more visible, even if there is `pg.dropped.2` that clearly refers to such column. In the above example it is easy enough: only one column has been dropped in a single user instance, however in a more concurrent system it is hard to get track about the information related to dropped attributes. For more information about the dropped columns, please see [my previous article about why PostgreSQL does not reclaim disk space on column drop](https://fluca1978.github.io/2020/02/09/PostgreSQLDROPCOlumn.html).


## What about `VACUUM`?

The `VACUUM FULL` command totally rewrites a table, therefore this means that every information about transactions that have "touched" systsem catalogs are updated by a newer transaction. **This does not mean that `VACUUM` is a transactional command**, rather it happen to do a `CREATE TABLE` pretty much as we did manually.

<br/>
<br/>
```sql
testdb=> VACUUM FULL ddl_test;                                                                                 
VACUUM
testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , current_timestamp - pg_xact_commit_timestamp( xmin ) as modified_since
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
 transaction_before | it_was_transaction_number | modified_since  |  table   
--------------------+---------------------------+-----------------+----------
                  1 |                2359180423 | 00:00:02.615678 | ddl_test
(1 row)

testdb=> SELECT array_agg( attname ) as columns,  
                current_timestamp - pg_xact_commit_timestamp( xmin ) as when 
         FROM pg_attribute
         WHERE attrelid = 'ddl_test'::regclass
         GROUP BY pg_xact_commit_timestamp( xmin );
                columns                 |      when       
----------------------------------------+-----------------
 {tableoid,cmax,xmax,cmin,xmin,ctid,pk} | 00:50:03.972343
 {ttt}                                  | 00:27:46.530726
 {tt}                                   | 00:26:10.077933
 {........pg.dropped.2........}         | 00:25:36.788619

```
<br/>
<br/>

It is interesting to note an apparent inconsistency: the table has been modified 2 seconds ago while the columns have been touched between 25 and 50 minutes ago. How is that possible? Well, `VACUUM FULL` has rewritten the table but metadata about columns did not change.
<br/>
In short, this is an indicator about `VACUUM FULL` execution: if the change time of a table is earlier than that of its columns probably vacuum ran. The correct way to know *when* `VACUUM FULL` run is to inspect appropriate catalogs like `pg_stat_user_tables`. In any case, combining these information help understanding what happened into the system.


<br/>
<br/>
Let's see something about `VACUUM`:


<br/>
<br/>
```sql
testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , current_timestamp - pg_xact_commit_timestamp( xmin ) as modified_since
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+----------------
transaction_before        | 11
it_was_transaction_number | 2359180423
modified_since            | 00:20:41.953205
table                     | ddl_test


testdb=> VACUUM ddl_test;
VACUUM

testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , current_timestamp - pg_xact_commit_timestamp( xmin ) as modified_since
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+----------------
transaction_before        | 11
it_was_transaction_number | 2359180423
modified_since            | 00:20:51.272209
table                     | ddl_test

```
<br/>
<br/>

The result is that `pg_class` is unchanged, with regard to the transaction that generated the tuple.
<br/>
Why?
<br/>
Since `VACUUM` is a command that cannot be run within a transaction, it cannot be considered in the described workflow, therefore it is like *an invisible command (with regard to transactions)*.



## What about `ANALYZE`?

**Unlike `VACUUM`, the command `ANALYZE` can be run in a transaction**, and this is clearly shown by the `age` result increasing by one:

<br/>
<br/>
```sql
testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , current_timestamp - pg_xact_commit_timestamp( xmin ) as modified_since
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+----------------
transaction_before        | 14
it_was_transaction_number | 2359180423
modified_since            | 01:37:54.483495
table                     | ddl_test


testdb=> ANALYZE ddl_test;
ANALYZE

testdb=> SELECT age( xmin ) as transaction_before
                , xmin as it_was_transaction_number
                , current_timestamp - pg_xact_commit_timestamp( xmin ) as modified_since
                , relname as table
         FROM pg_class
         WHERE relkind = 'r'
         AND relname   = 'ddl_test';
-[ RECORD 1 ]-------------+----------------
transaction_before        | 15
it_was_transaction_number | 2359180423
modified_since            | 01:38:05.267443
table                     | ddl_test

```
<br/>
<br/>

What is not changing in the above example is the transaction that generated the tuple in `pg_class`: it is always `2359180423`, before and after the `ANALYZE` command (that did run in a transaction).
<br/>
Why?
<br/>
Well, `ANALYZE` *hits* another table: `pg_statistic`. Such table is the root of all statistical information like `pg_stat_user_tables` and friends, and is the one updated by `ANALYZE`. This can be clearly inspected with a similar query:

<br/>
<br/>
```sql
testdb=# SELECT xmin, age( xmin ), staattnum 
         FROM pg_statistic 
         WHERE starelid = 'ddl_test'::regclass;
    xmin    | age | staattnum 
------------+-----+-----------
 2359180437 |   1 |         1
 2359180437 |   1 |         3
 2359180437 |   1 |         4

```
<br/>
<br/>

Please note that the query has been run as a superuser, because of the need of privileges. The result set is made of three rows because there are three "active" (i.e., not dropped) columns within the table, and all of them has been modified (from a statistic point of view) by `ANALYZE`, that ran in transaction `2359180437` that is now one transaction far (i.e., it was the previous transaction).

# Conclusions

Keeping track of commit timestamps could be useful for database introspection, at least to get a glance at **when** things changed.
<br/>
The same trick can also be used against regular table tuples, to get an idea of when a tuple appeared in that form in the table.
<br/>
However, this is not a very good approach, and something much more complex can be built like using already mentioned event triggers.
<br/>
*But hey, this is PostgreSQL: you can extend it in pretty much any direction!* 
