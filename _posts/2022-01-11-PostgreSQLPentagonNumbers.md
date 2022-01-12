---
layout: post
title:  "Pentagon numbers"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
A couple of different solutions to an interesting problem.

# Pentagon numbers

Since a few weeks, I tend to implement some of the tasks for the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} into PostgreSQL specific code. One interesting problem has been this week task 2 of the [Challenge 147](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0147/){:target="_blank"}: finding out a couple of **pentagon numbers** that have simultaneously a sum and a diff that is another pentagon number.
<br/>
In this post, I discuss two possible solutions to the task.
<br/>

## What is a Pentagon Number?

A **pentagon number** is defined as the value of the expression `n * ( 3 * n - 1 ) / 2`, therefore the pentagon number corresponding to `3` is `12`.
<br/>
The task required to find out a couple of pentagon numbers so that:

<br/>
<br/>

``` mathematica
P(n1) + P(n2) = P(x)
P(n1) - P(n2) = P(y)
```
<br/>
<br/>

It does not matter what `x` and `y` are, but `n1` and `n2` must be pentagon numbers and both their sum and diff must be pentagon numbers too.


## The first approach: a record based function

The first solution I came with was inspired by the solution I provided in Raku, and is quite frankly a kind of *record-based* approach.
<br/>
Firs of all, I define an `IMMUTABLE` function named `f_pentagon` that computes the pentagon number value starting from a given number, so that `f_pentagon( 3 )` returns `12`. Why do I need a function? Because I want to implement a table with a stored virtual column to keep track of numbers and their pentagon values.
<br/>
For that reason, I created a `pentagons` table with a generic `n` column that represents the starting value and the `p` column that represents the computed pentagon value.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_pentagon( n bigint )
RETURNS bigint
AS
$CODE$
        SELECT ( n * ( 3 * n - 1 ) / 2 );
$CODE$
LANGUAGE sql
IMMUTABLE;


DROP TABLE IF EXISTS pentagons;
CREATE TABLE pentagons
(
        n bigint
        , p bigint GENERATED ALWAYS AS ( f_pentagon( n ) ) STORED
);



INSERT INTO pentagons( n )
SELECT generate_series( 1, 5000 );


```
<br/>
<br/>


I inserted into the table `5000` records because I know, from the Raku solution, that what I'm looking for is within such range of values. It is, of course, possible to increase that limit to find out other values.
<br/>
The table content looks therefore like the following:


<br/>
<br/>

``` sql
testdb=> select * from pentagons limit 10;
 n  |  p
----+-----
  1 |   1
  2 |   5
  3 |  12
  4 |  22
  5 |  35
  6 |  51
  7 |  70
  8 |  92
  9 | 117
 10 | 145

```
<br/>
<br/>


Now it is possible to implement a function, named `f_pentagon_pairs` that seeks the above table searching for the required values. The table returns a `TABLE`, even if only one row will be returned, but since I want to output multiple values, I decided to implement it as a row level returning function. In particular, the returned information is:
- `n1` is the first number;
- `n2` is the second number;
- `s` is the sum of the pentagons, that is `P(n1) + P(n2)`;
- `d` is the difference of the pentagons, that is `abs( P(n1) - P(n2) )`;
- `ps` is the number which pentagon corresponds to the sum of the two pentagons, that is `P(ps) = P(n1) + P(n2)`;
- `pd` is the number which pentagon corresponds to the difference of the two pentagons, that is `P(pd) = abs( P(n1) - P(n2) )`;

<br/>
The function is the following one:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
f_pentagons_pairs()
RETURNS TABLE ( n1 bigint, n2 bigint,  s bigint, d bigint, ps bigint, pd bigint )
AS $CODE$
DECLARE
        current_tuple pentagons%rowtype;
        other_tuple   pentagons%rowtype;
        fnd           int := 0;
BEGIN

        FOR current_tuple IN SELECT * FROM pentagons ORDER BY n LOOP
            SELECT *
            INTO other_tuple
            FROM pentagons pp
            WHERE EXISTS(
                  SELECT *
                  FROM pentagons ps
                  WHERE ps.p = current_tuple.p + pp.p
                  )
           AND EXISTS (
               SELECT *
               FROM pentagons ps
               WHERE ps.p = abs( current_tuple.p - pp.p )
           );


           IF FOUND THEN
              SELECT current_tuple.n
                     , other_tuple.n
                     , current_tuple.p + other_tuple.p
                     , abs( current_tuple.p - other_tuple.p )
                     , p1.n
                     , p2.n
              INTO n1, n2, s, d, ps, pd
              FROM pentagons p1, pentagons p2
              WHERE p1.p = current_tuple.p + other_tuple.p
              AND   p2.p = abs( current_tuple.p - other_tuple.p );

              RAISE INFO 'P(%) + P(%) = P(%) =  %',
                         n1, n2, ps, s;

             RAISE INFO 'P(%) - P(%) = P(%) =  %',
                        n1, n2, pd, d;


              fnd := fnd + 1;
              RETURN NEXT;
              RETURN;
           END IF;

        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

It is quite simple to understand:
- it performs a *one-record-at-time* loop placing every row of `pentagons` into `current_tuple`;
- it searches for an`other_tuple` in `pentagons` so that the sum and the difference `EXISTS` in `pentagons` at the very same time. This means that the `other_tuple` and `current_tuple` lead to a sum and a difference that is still another pentagon number;
- when such tuple is `FOUND`, the output tuple is built.

<br/>
In order to get the *reverse* values that lead to the sum and difference, I do another double join with `pentagons` to get out the result.
<br/>
The `RAISE` instructions are placed only to provide a textual representation of the expressions.
<br/>
Launching the function on a very little virtual machine, busy in doing other stuff, results in:



<br/>
<br/>

``` sql
testdb=> SELECT * FROM f_pentagons_pairs();
INFO:  P(1020) + P(2167) = P(2395) =  8602840
INFO:  P(1020) - P(2167) = P(1912) =  5482660
  n1  |  n2  |    s    |    d    |  ps  |  pd
------+------+---------+---------+------+------
 1020 | 2167 | 8602840 | 5482660 | 2395 | 1912
(1 row)

Time: 3346,886 ms (00:03,347)
```
<br/>
<br/>



## A CTE Approach

Is there the need for the `pentagons` table? Uhm...it is possible to materialize the same set of data with a recursive CTE.
And, therefore, it is possible to move the query at the outer level so that there is no need to perform a record-by-record scan.
After all, SQL is a set based language!


<br/>
<br/>

``` sql
WITH RECURSIVE pentagons( n, p )
AS
(
        SELECT 1 AS n
               , f_pentagon( 1 ) AS p

UNION
        SELECT p.n + 1
               , f_pentagon( p.n + 1 )
        FROM pentagons p
        WHERE p.n < 5000
)

SELECT format( '%s, %s', l.n, r.n ) AS pentagon_pairs
FROM pentagons l, pentagons r
WHERE EXISTS(
      SELECT *
      FROM pentagons ps
      WHERE ps.p = l.p + r.p
      )
AND EXISTS (
    SELECT *
    FROM pentagons ps
    WHERE ps.p = abs( l.p - r.p )
    )
;

```
<br/>
<br/>

The query executes in a little less time than the approach using the table and the record-based function:

<br/>
<br/>

``` sql
 pentagon_pairs
----------------
 1020, 2167
 2167, 1020
(2 rows)

Time: 5820,066 ms (00:05,820)

```
<br/>
<br/>


There are some details that are harder to tune with the CTE approach, most notably the reverse lookup of the resulting base numbers and the exclusion of the duplicated row. However, it is possible to tune it to your needs.
<br/>
Why is the CTE approach require more time than the function approach? Well, even if the times are similar, the function *terminates* as soon as it finds a solution, while the CTE does not, and therefore *scans* the whole dataset.



## Plans!

Timing is not as much difference as it could seem at glance, and effectively the two approaches are comparable with regard to performances.
The execution plans are, of course, a lot more different since the function approach works as a black box:


<br/>
<br/>

``` sql
testdb=> EXPLAIN ANALYZE SELECT * FROM f_pentagons_pairs();
INFO:  P(1020) + P(2167) = P(2395) =  8602840
INFO:  P(1020) - P(2167) = P(1912) =  5482660
                                                        QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
 Function Scan on f_pentagons_pairs  (cost=0.25..10.25 rows=1000 width=48) (actual time=4754.081..4754.082 rows=1 loops=1)
 Planning Time: 0.047 ms
 Execution Time: 4856.988 ms
(3 rows)

Time: 4909,165 ms (00:04,909)
testdb=> EXPLAIN ANALYZE WITH RECURSIVE pentagons( n, p )
AS
(
        SELECT 1 AS n
               , f_pentagon( 1 ) AS p
UNION
        SELECT p.n + 1
               , f_pentagon( p.n + 1 )
        FROM pentagons p
        WHERE p.n < 5000
)
SELECT format( '%s, %s', l.n, r.n ) AS pentagon_pairs
FROM pentagons l, pentagons r
WHERE EXISTS(
      SELECT *
      FROM pentagons ps
      WHERE ps.p = l.p + r.p
      )
AND EXISTS (
    SELECT *
    FROM pentagons ps
    WHERE ps.p = abs( l.p - r.p )
    )
;
                                                          QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------
 Hash Semi Join  (cost=5.57..30.98 rows=23 width=32) (actual time=2659.801..10415.225 rows=2 loops=1)
   Hash Cond: (abs((l.p - r.p)) = ps_1.p)
   CTE pentagons
     ->  Recursive Union  (cost=0.00..3.56 rows=31 width=12) (actual time=0.002..21.821 rows=5000 loops=1)
           ->  Result  (cost=0.00..0.01 rows=1 width=12) (actual time=0.001..0.001 rows=1 loops=1)
           ->  WorkTable Scan on pentagons p  (cost=0.00..0.29 rows=3 width=12) (actual time=0.000..0.000 rows=1 loops=5000)
                 Filter: (n < 5000)
                 Rows Removed by Filter: 0
   ->  Hash Semi Join  (cost=1.01..25.63 rows=149 width=24) (actual time=361.447..10156.895 rows=5341 loops=1)
         Hash Cond: ((l.p + r.p) = ps.p)
         ->  Nested Loop  (cost=0.00..20.15 rows=961 width=24) (actual time=0.005..6323.391 rows=25000000 loops=1)
               ->  CTE Scan on pentagons l  (cost=0.00..0.62 rows=31 width=12) (actual time=0.002..0.671 rows=5000 loops=1)
               ->  CTE Scan on pentagons r  (cost=0.00..0.62 rows=31 width=12) (actual time=0.000..0.546 rows=5000 loops=5000)
         ->  Hash  (cost=0.62..0.62 rows=31 width=8) (actual time=354.101..354.101 rows=5000 loops=1)
               Buckets: 8192 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 260kB
               ->  CTE Scan on pentagons ps  (cost=0.00..0.62 rows=31 width=8) (actual time=0.001..0.405 rows=5000 loops=1)
   ->  Hash  (cost=0.62..0.62 rows=31 width=8) (actual time=255.920..255.920 rows=5000 loops=1)
         Buckets: 8192 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 260kB
         ->  CTE Scan on pentagons ps_1  (cost=0.00..0.62 rows=31 width=8) (actual time=0.004..49.709 rows=5000 loops=1)
 Planning Time: 0.195 ms
 Execution Time: 10431.113 ms
(21 rows)

Time: 10509,619 ms (00:10,510)
```
<br/<
<br/>

Changing the CTE between `MATERIALIZED` and `NOT MATERIALIZED` does not produce any sensible change, of course.
<br/>
Creating an index on `pentagons(p)` makes the function approach a little faster, but not very much faster since it is used only in the final part of the function.


## A query only approach

Having the `pentagons` table in place, it is possible to use it as the materialization of the CTE, thus pushing the query out of the function and not within the CTE:


<br/>
<br/>

``` sql
testdb=> SELECT format( '%s, %s', l.n, r.n ) AS pentagon_pairs
FROM pentagons l, pentagons r
WHERE EXISTS(
      SELECT *
      FROM pentagons ps
      WHERE ps.p = l.p + r.p
      )
AND EXISTS (
    SELECT *
    FROM pentagons ps
    WHERE ps.p = abs( l.p - r.p )
    )
;
 pentagon_pairs
----------------
 1020, 2167
 2167, 1020
(2 rows)

Time: 5024,468 ms (00:05,024)

```
<br/>
<br/>


It is possible to push some `LIMIT 1` into the subqueries, so to force them to terminate as soon as a match is found, and this slightly improves the speed of the whole query:


<br/>
<br/>

``` sql
testdb=> SELECT format( '%s, %s', l.n, r.n ) AS pentagon_pairs
FROM pentagons l, pentagons r
WHERE EXISTS(
      SELECT *
      FROM pentagons ps
      WHERE ps.p = l.p + r.p LIMIT 1
      )
AND EXISTS (
    SELECT *
    FROM pentagons ps
    WHERE ps.p = abs( l.p - r.p ) LIMIT 1
    )
;
 pentagon_pairs
----------------
 1020, 2167
 2167, 1020
(2 rows)

Time: 4328,603 ms (00:04,329)

```
<br/>
<br/>

The number of rows on the table is, however, too small for triggering the usage of the index, even forcing an `ORDER BY`.
Even an including index, that could cover all the columns, will not help in this case.


# Conclusiops

*There is more than one way to do it!*
<br/>
No, sorry, this is not Perl, but PostgreSQL! However, given a specific problem, PostgreSQL provides a lot of fun and tools to solve a task.
