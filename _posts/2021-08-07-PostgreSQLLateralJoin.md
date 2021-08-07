---
layout: post
title:  "A simple example of LATERAL use" 
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
How `LATERAL` can help to solve problems...

# A simple example of LATERAL use

A few days ago I found a question by a user on Facebook: *how to select events from a table where they are no more than 10 minutes one from another?*
<br/>
My first answer was related to `LATERAL`, and this post I try to represent with an example how I understood and could solve the above question.

<br/>
<br/>
First of all, let's build an *events* table, where each row has a timestamp.

<br/>
<br/>
```sql
testdb=> CREATE TABLE events(
  pk int generated always as identity
  , event text
  , ts timestamp default CURRENT_TIMESTAMP
  , PRIMARY KEY( pk )
);

CREATE TABLE
```
<br/>
<br/>
Now, let's populate the table with "random" data so that there are events spread in a range of `2 minutes` each:

<br/>
<br/>
```sql
testdb=> insert into events( event, ts )
select 'event #' || v, current_timestamp - ( ( v * 2 ) || ' minutes' )::interval
from generate_series( 1, 100 ) v;

INSERT 0 100
```
<br/>
<br/>

Having setup the table and the data, how can we relate every tuple with other events that are not outside a ten minutes window? `LATERAL` comes to the rescue.

<br/>
<br/>
```sql
testdb=> SELECT e1.pk, e1.event, e2.*  
         FROM events e1, 
         LATERAL ( SELECT pk, event, ts - e1.ts as time_elapsed 
                   FROM events 
                   WHERE pk <> e1.pk AND e1.ts - ts <= '10 minutes'::interval ) e2 
        ORDER BY e1.pk LIMIT 20;
        
 pk  |  event   | pk  |  event   | time_elapsed 
-----+----------+-----+----------+--------------
 501 | event #1 | 502 | event #2 | -00:02:00
 501 | event #1 | 503 | event #3 | -00:04:00
 501 | event #1 | 504 | event #4 | -00:06:00
 501 | event #1 | 505 | event #5 | -00:08:00
 501 | event #1 | 506 | event #6 | -00:10:00
 502 | event #2 | 501 | event #1 | 00:02:00
 502 | event #2 | 503 | event #3 | -00:02:00
 502 | event #2 | 504 | event #4 | -00:04:00
 502 | event #2 | 505 | event #5 | -00:06:00
 502 | event #2 | 506 | event #6 | -00:08:00
 502 | event #2 | 507 | event #7 | -00:10:00
 503 | event #3 | 501 | event #1 | 00:04:00
 503 | event #3 | 502 | event #2 | 00:02:00
 503 | event #3 | 504 | event #4 | -00:02:00
 503 | event #3 | 505 | event #5 | -00:04:00
 503 | event #3 | 506 | event #6 | -00:06:00
 503 | event #3 | 507 | event #7 | -00:08:00
 503 | event #3 | 508 | event #8 | -00:10:00
 504 | event #4 | 501 | event #1 | 00:06:00
 504 | event #4 | 502 | event #2 | 00:04:00
(20 rows)

```
<br/>
<br/>
Let's disassemble the query and see how it works.
The subquery selects all the tuples that are within a `10 minutes` range and that are different from the query the system is currently evaluating (i.e., `e1.pk`). But usually a subquery is evaluated once for the outer query, but note that the subquery is prefixed with `LATERAL` that, in simple words, means *evaluate the subquery for every row of the outer result set*. This means that the `LATERAL` subquery can access the outer query row, and can "reason" about its own result set.
<br/>
An important thing to keep in mind while dealing with `LATERAL` is that the subquery must be referenced with an alias, in my case `e2`. Please note that within the `LATERAL` subquery I do compute the time difference between the timestamp of the outer tuple and the one of the inner result set, and as you can see from the output column `time_elapsed` every row differs by 2 minutes, that is how we generated the rows.
<br/>
<br/>
What happens if you don't use `LATERAL`? Well, you cannot reference the `e1` outer tuple, that is there is no way for a subquery to cross-reference something outside of its scope:

<br/>
<br/>
```sql
testdb=> SELECT e1.pk, e1.event, e2.*  
         FROM events e1,  
         ( SELECT pk, event, ts - e1.ts as time_elapsed 
           FROM events WHERE pk <> e1.pk AND e1.ts - ts <= '10 minutes'::interval ) e2 
         ORDER BY e1.pk LIMIT 20;
ERROR:  invalid reference to FROM-clause entry for table "e1"
LINE 1: ..., e2.*  FROM events e1,  ( SELECT pk, event, ts - e1.ts as t...
                                                             ^
HINT:  There is an entry for table "e1", but it cannot be referenced from this part of the query.

```
<br/>
<br/>
As you can see, PostgreSQL clearly states that you cannot refer to `e1` (the outer tuple) from within the scope of the subquery.

## `LATERAL` Joins

It is, of course, possible to use `LATERAL` in a join, and in this case the above query can be rewritten as:

<br/>
<br/>
```sql
testdb=> SELECT e1.pk, e1.event, e2.*  
         FROM events e1 JOIN  LATERAL 
            ( SELECT pk, event, ts - e1.ts as time_elapsed 
              FROM events WHERE pk <> e1.pk AND e1.ts - ts <= '10 minutes'::interval ) e2 
        ON true  
        ORDER BY e1.pk LIMIT 20;
 pk  |  event   | pk  |  event   | time_elapsed 
-----+----------+-----+----------+--------------
 501 | event #1 | 502 | event #2 | -00:02:00
 501 | event #1 | 503 | event #3 | -00:04:00
 501 | event #1 | 504 | event #4 | -00:06:00
 501 | event #1 | 505 | event #5 | -00:08:00
 501 | event #1 | 506 | event #6 | -00:10:00
 502 | event #2 | 501 | event #1 | 00:02:00
 502 | event #2 | 503 | event #3 | -00:02:00
 502 | event #2 | 504 | event #4 | -00:04:00
 502 | event #2 | 505 | event #5 | -00:06:00
 502 | event #2 | 506 | event #6 | -00:08:00
 502 | event #2 | 507 | event #7 | -00:10:00
 503 | event #3 | 501 | event #1 | 00:04:00
 503 | event #3 | 502 | event #2 | 00:02:00
 503 | event #3 | 504 | event #4 | -00:02:00
 503 | event #3 | 505 | event #5 | -00:04:00
 503 | event #3 | 506 | event #6 | -00:06:00
 503 | event #3 | 507 | event #7 | -00:08:00
 503 | event #3 | 508 | event #8 | -00:10:00
 504 | event #4 | 501 | event #1 | 00:06:00
 504 | event #4 | 502 | event #2 | 00:04:00
(20 rows)
```
<br/>
<br/>


# Conclusions

`LATERAL` is a very powerful SQL operator in PostgreSQL, and can help solving problems you would normally solve by means of cursors and iterations.
