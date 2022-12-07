---
layout: post
title:  "Using table aliases in UPSERTs"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A simple way to achieve a "counter-like" using UPSERTs.

# Using table aliases in UPSERTs

PostgreSQL has had the *UPSERT* feature since a while, now somehow overtaken by the [`MERGE`](https://www.postgresql.org/docs/15/sql-merge.html){:target="_blank"} command.
<br/>
One interesting feature of `UPSERT` is that it can quickly help you to implement a *counter-like* approach, but often you need to use table aliasing in order for the feature to be able to accumulate results.
<br/>
Allow me to explain with a very simple example: we want to count the occurrencies of every letter into a bunch of text, and we want it to be able to accumulate between different calls.
One possible solution is to use a table, even a `TEMPORARY` one, to store the letter and a counter, and then populate such table.

With an *UPSERT* the thing reduces to:

<br/>
<br/>
```sql
testdb=> create temp table letters( l char primary key, c int default 0 ) on commit delete rows;
CREATE TABLE

testdb=> begin;
BEGIN

testdb=*> with flow as ( select l, count(*) as n
               from regexp_split_to_table( 'fhcgfeettrrzz', '' ) l
               group by l )
insert into letters as mem  -- table alias
select l, n
from flow
on conflict( l )
do update set c = mem.c + excluded.c; -- accumulate !
INSERT 0 8

-- repeat with different sources of text

testdb=*> table letters;
 l | c
---+----
 a | 12
 b | 20
 r |  2
 z |  2
 g |  1
 c |  1
 t |  2
 h |  1
 f | 18
 e | 18
(10 rows)

```
<br/>
<br/>



The `flow` CTE is simulating our flow of letters, and provides an instant counting of the occurencies. We want such counting to accumulate into the `letters` table, but we want to use the *UPSERT* feature.
When a conflict is found, over the primary key `l`, the `INSERT` degenrates into an `UPDATE` and the `c` counter must be increased by the value of the `excluded` tuple counter.
Here arise the problem: how can we refer to the current value of the counter? **We need a table aliasing**, in our case `mem` to refer to the current tuple.
<br/>
If we omit the `mem.c` from the update statement, the query will refuse to work because there is ambiguity on which tuple column we are referring to:


<br/>
<br/>
```sql
testdb=*> with flow as ( select l, count(*) as n
               from regexp_split_to_table( 'fhcgfeettrrzz', '' ) l
               group by l )
insert into letters as mem
select l, n
from flow
on conflict( l )
do update set c = c + excluded.c
;
ERROR:  column reference "c" is ambiguous
LINE 8: do update set c = c + excluded.c

```
<br/>
<br/>

Note that the column name on the left of the assignement does not need to be disambiguated, since it is clear that we are referring to the conflicting tuple we tried to insert.
