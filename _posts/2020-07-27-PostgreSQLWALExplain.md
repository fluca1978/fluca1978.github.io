---
layout: post
title:  "PostgreSQL 13 Explain now includes WAL information"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
The upcoming version of PostgreSQL now includes new information in the EXPLAIN output.

# PostgreSQL 13 Explain now includes WAL information

The upcoming PostgreSQL 13 includes a lot of new features, as a very consolidated habit in every release. One interesting feature among the others is that [`EXPLAIN` now supports a new `WAL` option](https://www.postgresql.org/docs/13/sql-explain.html){:target="_blank"} (that requires `ANALYZE` to be set).
<br/>
This new `WAL` feature allows `EXPLAIN` to provide information about the generated amount of WAL traffic.
It is quite simple to see it in action:

<br/>
<br/>
```sql
testdb=> CREATE TABLE foo( i int generated always as identity, t text );

testdb=> EXPLAIN ( ANALYZE, WAL, FORMAT yaml ) 
         INSERT INTO foo( t )
         SELECT  md5( v::text )
         FROM generate_series( 1, 300000 ) v;  
         
                QUERY PLAN                
------------------------------------------
 - Plan:                                 +
     Node Type: "ModifyTable"            +
     Operation: "Insert"                 +
     Parallel Aware: false               +
     Relation Name: "foo"                +
     Alias: "foo"                        +
     Startup Cost: 0.00                  +
     Total Cost: 6000.00                 +
     Plan Rows: 300000                   +
     Plan Width: 36                      +
     Actual Startup Time: 508.168        +
     Actual Total Time: 508.168          +
     Actual Rows: 0                      +
     Actual Loops: 1                     +
     WAL Records: 309091                 +
     WAL FPI: 0                          +
     WAL Bytes: 28500009                 +
     ...
```
<br/>
<br/>
As you can see, the output of `EXPLAIN` now includes three new nodes:
- `WAL Records`, as the name suggests, is the number of WAL records inserted into the logs;
- `WAL FPI` is the number of the *F*ull *P*age *I*mages inserted into the WALs;
- `WAL bytes` is the amount of *traffic* generated towards the WAL logs.

The number of WAL records does not match exactly the number of tuple inserted by the query, clearly, but it is equal or greater. You can check this with a small number of inserts:

<br/>
<br/>
```sql
testdb=> EXPLAIN ( ANALYZE, WAL, FORMAT yaml ) 
         INSERT INTO foo( t )
         SELECT  md5( v::text )
         FROM generate_series( 1, 3 ) v;
         
                QUERY PLAN                
------------------------------------------
 - Plan:                                 +
     Node Type: "ModifyTable"            +
     Operation: "Insert"                 +
     ...
     WAL Records: 3                      +
     WAL FPI: 0                          +
     WAL Bytes: 276                      +
     ...
``**
<br/>
<br/>

I think this can help in understanding the amount of traffic passing thru the WALs, and therefore helping in configuring properly also the checkpoint related settings in a more aggressive way.
<br/>
**[`auto_explain`](https://www.postgresql.org/docs/13/auto-explain.html){:target="_blank"} does support WAL information dump too**, via the special configuration parameter `auto_explain.log_wal`.
<br/>
<br/>
