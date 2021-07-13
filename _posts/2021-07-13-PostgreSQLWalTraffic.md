---
layout: post
title:  "How much data goes into the WALs?" 
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
What is the amount of traffic generated in the Write Ahead Logs?

# How much data goes into the WALs?

PostgreSQL exploits the *Write Ahead Log*s (WALs) to make data changes persistent: whenever you `COMMIT` (implicitly or explicitly) a work, the data is stored in the WALs before it phisically hits the table it belongs to.
<br/>
There are different advantages in this approach, most notably performances and the ability to survive a crash.
<br/>
And one beautiful thing about PostgreSQL is that it provides you all the tools to follow, study and understand what it is happening under the hood. With regard to the WALs, there are few `pg_wal_xxx` functions that can be exploited to get a clue about what is happening in the WALs.
<br/>
In this post I'm going to use mainly:
- `pg_current_wal_lsn()` that provides the current offset within the WAL stream where the *next thing* will happen. Such offset in the WAL stream is called **Log Sequence Number** or *LSN* for short;
- `pg_walfile_name()` that given a Log Sequence Number (*LSN*) provides you the name of the WAL file, in the `pg_wal` directory, that contains the WAL location.


<br/>
<br/>
It is worth spending a little time to explain what LSNs are.
<br/>
PostgreSQL organizes the WALs into files large `16 MB` each (you can change this setting, but assume you will not). Every time a WAL file is full, that is it contains `16 MBG` of *valid WAL data*, PostgreSQL produces a new file (or recycles a no more used one).
<br/>
The database must know exactly when things happened during the *history of transactions*, and this means it must be able to point to a location into the WAL files to clearly identify a transaction, or a statement, or something else. This location is expressed a *Log Sequence Number*, something that points the server to an offset within the WAL stream. 
<br/>
Therefore, when you execute an SQL statement, the database stores the result of the statement into the WALs at the position indicated by the current log sequence number, and the next statement will happen at a different log sequence number.
<br/>
Log sequence numbers have the form of `AA/BBxxxxxx` where `AA` and `BB` can be used to identify the WAL file on disk (knowing the current timeline). In fact, usually the WAL file that contains a log sequence number is named as `000000<timeline>000000AA000000BB`. As an example, if the LSN is `16/70D22618` the corresponding file on disk is `000000070000001600000070` (given the timeline number `7`). This rule of thumb is not always true, since the LSN could be near the end of the WAL file, or even on the beginning of the new one, but you get the idea.
The remaining part, represented by `xxxxxx` is the offset within the WAL file to find the position of the LSN.
<br/>
PostgreSQL has a dedicated data type, `pg_lsn`, to store information about a Log Sequence Number. You can apply  operators to `pg_lsn`, for example to do a difference between two values, and PostgreSQL will show you the result as a `numeric` value.
<br/>
Now that is clear what a LSN is and how it relates to the WAL files on disk, let's see how it is possible to get the amount of data written in the WALs with regards to the amount of data written to a table. In the following examples I'm using a server 13.3 with only me running queries, so numbers are effectively related only to my experimentations.

## An example with a normal table

Let's create a very simple table:

<br/>
<br/>
```sql
testdb=> create table logged_table( t text );
CREATE TABLE
```
<br/>
<br/>
Now let's do a bulk insert, and check the current *Log Seqeuence Number* before and after the insertion of one million tuples:


<br/>
<br/>
```sql
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), pg_current_wal_lsn(), pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000070 | 16/70D22618        | 0 bytes
(1 riga)

testdb=> insert into logged_table( t ) 
         select 'logged ' || v from generate_series(1, 1000000 ) v;
INSERT 0 1000000

testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000075 | 16/752611C8        | 42 MB
(1 riga)

testdb=> select pg_size_pretty( '16/752611C8'::pg_lsn - '16/70D22618'::pg_lsn );
 pg_size_pretty 
----------------
 69 MB
(1 riga)

```
<br/>
<br/>

As you can see, generating `42 MB` of **real table data** implied the generation of `69 MB` of WAL data. Why there is more data in the WALs than in the actual table? Because the WAL records must keep links to themselves, checksum and a lot of other data that can be used by PostgreSQL by replication and crash recovery.


## Using an unlogged table

Let's now start over, transforming the table as `UNLOGGED`, so that it is not going to hit the WALs.

<br/>
<br/>
```sql
testdb=> truncate table logged_table ;
TRUNCATE TABLE
testdb=> alter table logged_table set unlogged;
ALTER TABLE
testdb=> alter table logged_table rename to unlogged_table;
ALTER TABLE

```
<br/>
<br/>

Replay the same above insertion of one million tuples and see what happens to the WALs:


<br/>
<br/>
```sql
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), pg_current_wal_lsn(), pg_size_pretty( pg_relation_size( 'unlogged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000075 | 16/75285AD0        | 0 bytes
(1 riga)

testdb=> insert into unlogged_table( t ) 
         select 'logged ' || v from generate_series(1, 1000000 ) v;
INSERT 0 1000000

testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'unlogged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000075 | 16/75285B30        | 42 MB
(1 riga)

testdb=> select pg_size_pretty( '16/75285B30'::pg_lsn - '16/75285AD0'::pg_lsn );
 pg_size_pretty 
----------------
 96 bytes

```
<br/>
<br/>

As you can see, the table has grown by the same size of the previous example, that is `42 MB` of real data. This time however, the WAL records have not grown, except for a very little amount of `96 bytes` of roomkeeping datata.


## Going back to a logged table

What happens if the table comes back as `LOGGED`?

<br/>
<br/>
```sql
testdb=> alter table unlogged_table rename to logged_table;
ALTER TABLE
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000075 | 16/75295120        | 42 MB
(1 riga)

testdb=> alter table logged_table set logged;
ALTER TABLE
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000079 | 16/7978EFF0        | 42 MB
(1 riga)

testdb=> select pg_size_pretty( '16/7978EFF0'::pg_lsn - '16/75295120'::pg_lsn );
 pg_size_pretty 
----------------
 69 MB
(1 riga)

```
<br/>
<br/>
As you can see, setting the table from `UNLOGGED` to `LOGGED` generated pretty much the same amount of WAL traffice (i.e., `69 MB`) as in the original insert transaction.


## Add some fields

Let's add a couple of more fields to the table, to see what happens with regard to the traffic:

<br/>
<br/>
```sql
testdb=> alter table logged_table add column pk serial primary key;
ALTER TABLE
testdb=> alter table logged_table add column price numeric( 5, 2 ) default 0;
ALTER TABLE
testdb=> truncate logged_table ;
TRUNCATE TABLE

```
<br/>
<br/>

and now re-run our little benchmark (note that the added fields have default values):


<br/>
<br/>
```sql
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 00000007000000160000007D | 16/7DD16CA8        | 0 bytes
(1 riga)

testdb=> insert into logged_table( t ) 
         select 'logged ' || v from generate_series(1, 1000000 ) v;
INSERT 0 1000000
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) ),
                pg_size_pretty( pg_relation_size( 'logged_table_pkey' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty | pg_size_pretty
--------------------------|--------------------|----------------|----------------
 000000070000001600000086 | 16/86BD7110        | 50 MB          | 43 MB

(1 riga)

testdb=> select pg_size_pretty( '16/86BD7110'::pg_lsn - '16/7DD16CA8'::pg_lsn );
 pg_size_pretty 
----------------
 143 MB
(1 riga)

```
<br/>
<br/>
This time, as you can see, the table has grown about `20%` of its previous size, that is to `50 MB` of real data, but there is also the index (on the primary key column) to consider, and that is `43 MB`, for an overall total of `93 MB` of real data.
However, the WALs almost doubled their previous size, and still are larger than the size of the real data due to the structure of the records.


## Doing a rollback

What happens if the transaction does a rollback?
<br/>
WALs are managed as an *append-only* storage, so there will be WAL traffic. It is quite easy to experiment this:

<br/>
<br/>
```sql
testdb=> select pg_walfile_name( pg_current_wal_lsn() ),                        
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 00000007000000160000009E | 16/9EC3A680        | 0 bytes
(1 riga)

testdb=> begin;
BEGIN
testdb=*> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 00000007000000160000009E | 16/9EC3A680        | 0 bytes
(1 riga)

testdb=*> insert into logged_table( t )                                          
         select 'logged ' || v from generate_series(1, 1000000 ) v;
INSERT 0 1000000
testdb=*> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(),                              
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 0000000700000016000000A7 | 16/A7AFA000        | 50 MB
(1 riga)

testdb=*> rollback;
ROLLBACK
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 0000000700000016000000A7 | 16/A7AFAB50        | 50 MB
(1 riga)

testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000092 | 16/92D11AD8        | 0 bytes
(1 riga)

testdb=> begin;
BEGIN
testdb=*> select pg_walfile_name( pg_current_wal_lsn() ),                                          
                pg_current_wal_lsn(),                              
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 000000070000001600000092 | 16/92D11AD8        | 0 bytes
(1 riga)

testdb=*> insert into logged_table( t )                                          
         select 'logged ' || v from generate_series(1, 1000000 ) v;
INSERT 0 1000000
testdb=*> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(),                              
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 00000007000000160000009B | 16/9BBD0000        | 50 MB
(1 riga)

testdb=*> select pg_size_pretty( '16/9BBD0000'::pg_lsn - '16/92D11AD8'::pg_lsn );
 pg_size_pretty 
----------------
 143 MB
(1 riga)

testdb=*> rollback;
ROLLBACK
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty 
--------------------------|--------------------|----------------
 00000007000000160000009B | 16/9BBD1F40        | 50 MB
(1 riga)

testdb=> select pg_size_pretty( '16/9BBD1F40'::pg_lsn - '16/92D11AD8'::pg_lsn );
 pg_size_pretty 
----------------
 143 MB
(1 riga)
```
<br/>
<br/>

Before the transaction starts, the current LSN is `16/92D11AD8` and it remains unchanged until the transaction actually does some work. Before the `ROLLBACK` the LSN is `16/9BBD0000` and immediatly after the `ROLLBACK` the LSN moved forward to `16/9BBD1F40`. Therefore, *simply* issuing a `ROLLBACK` caused the WAL to increase about `8kB`.

## `pg_waldump`

The special command `pg_waldump`  provides information about WAL contents.
<br/>
**It is required to have the WALs** to inspect: as trivial as it could sound, you will not be able to *observe* your transaction if the database has executed a `CHECKPOINT` and has recycled the WAL segments (but you can archive them if you want to inspect *old* transactions).

<br/>
Let's play again our rollback transaction to get effective LSN numbers:

<br/>
<br/>
```sql
testdb=> truncate logged_table ;
TRUNCATE TABLE
testdb=> begin;
BEGIN
testdb=*> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) ),
                pg_size_pretty( pg_relation_size( 'logged_table_pkey' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty | pg_size_pretty 
--------------------------|--------------------|----------------|----------------
 0000000700000016000000C3 | 16/C3E18BB0        | 0 bytes        | 8192 bytes
(1 riga)

testdb=*> insert into logged_table( t ) values( 'a single record' );
INSERT 0 1
testdb=*> select pg_walfile_name( pg_current_wal_lsn() ),           
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) ),
                pg_size_pretty( pg_relation_size( 'logged_table_pkey' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty | pg_size_pretty 
--------------------------|--------------------|----------------|----------------
 0000000700000016000000C3 | 16/C3E18BB0        | 8192 bytes     | 16 kB
(1 riga)

testdb=*> rollback;
ROLLBACK
testdb=> select pg_walfile_name( pg_current_wal_lsn() ), 
                pg_current_wal_lsn(), 
                pg_size_pretty( pg_relation_size( 'logged_table' ) ),
                pg_size_pretty( pg_relation_size( 'logged_table_pkey' ) );
     pg_walfile_name      | pg_current_wal_lsn | pg_size_pretty | pg_size_pretty 
--------------------------|--------------------|----------------|----------------
 0000000700000016000000C3 | 16/C3E18D48        | 8192 bytes     | 16 kB
(1 riga)

testdb=> select pg_size_pretty( '16/C3E18D48'::pg_lsn - '16/C3E18BB0'::pg_lsn );
 pg_size_pretty 
----------------
 408 bytes
(1 riga)

```
<br/>
<br/>
Why inserting  asingle tuple this time? Because when using `pg_waldump` the system is going to produce a very verbose output and I don't want to mess with a ton of `INSERT`s.
<br/>
The above generated a very small amount of WAL traffic, `408 bytes` exactly. Let's inspect what is in the WALs by means of `pg_waldump`:


<br/>
<br/>
```shell
% sudo -u postgres /usr/pgsql-13/bin/pg_waldump -p $PGDATA/pg_wal  -s 16/C3E18BB0 -e 16/C3E18D48 -t 7 

rmgr: Heap        tx:    3562600, lsn: 16/C3E18BB0, prev 16/C3E18B50, desc: INSERT+INIT off 1 flags 0x00, blkref #0: rel 1663/89735/89935 blk 0
rmgr: Btree       tx:    3562600, lsn: 16/C3E18C00, prev 16/C3E18BB0, desc: NEWROOT lev 0, blkref #0: rel 1663/89735/89937 blk 1, blkref #2: rel 1663/89735/89937 blk 0
rmgr: Btree       tx:    3562600, lsn: 16/C3E18C68, prev 16/C3E18C00, desc: INSERT_LEAF off 1, blkref #0: rel 1663/89735/89937 blk 1
rmgr: Transaction tx:    3562600, lsn: 16/C3E18CA8, prev 16/C3E18C68, desc: ABORT 2021-07-13 04:59:03.235599 EDT

```
<br/>
<br/>
I've removed part of the information to better fit the screen size.
<br/>
The first entry on the top is the execution of the `INSERT` statement, followed by two entries that create the values in the index, and last there is the `ABORT`, that is the `ROLLBACK` statement.
<br/>
As you can see, every record has the clear indication of what LSN it is by means of the `lsn` field, as well as pointer to its previous record (i.e., the `previous` LSN offset). This way allows PostgreSQL to read the WAL stream from the end and go back in history to get the exact boundaries of a piece of work.
