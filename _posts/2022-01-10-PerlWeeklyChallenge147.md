---
layout: post
title:  "Perl Weekly Challenge 147: truncating pentagons"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 147: truncating pentagons

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 147](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0147/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg) and a [CTE only solution](#task2pgb)





<a name="task1"></a>
## PWC 147 - Task 1
The first task was about finding out a *left truncated prime number*, that is a prime number that is made by right prime numbers. In other words, given a prime number, and removing one at a time the leftmost digit, the right number is still a prime number itself.
<br/>
The task asked to find out the first 20 numbers, so I placed a default variable that decides the limit for generating the truncated prime numbers:


<br/>
<br/>
```raku
sub MAIN( Int $limit = 20 ) {

    my @primes;

    for 10 .. Inf -> $current {
        next if $current ~~ / 0 /;
        next if ! $current.is-prime;
        my @values.push: $current.comb[ $_ .. * - 1 ].join.Int for 0 ..^ $current.Str.chars;
        @primes.push: $current if @values.grep( *.is-prime ).elems == @values.elems;
        last if @primes.elems >= $limit;

    }

    @primes.join( "\n" ).say;
}

 ```
<br/>
<br/>

Since I need to truncate a number, it does not make sense to start from a single digit number, thus I start the loop from `10` to infinity.
I skip all numbers containing one or more zeros and that are not prime.
Then, I build a `@values` array made of all truncations of a number. Now, if I can find that all `ovalues` is made by primes, that is the number of primes is equal to the number of elements of the array, the number is a truncated prime, and thus can be added to the `@primes` array.
The loop is terminated as soon as the `$limit` of searched for numbers is found. And the rest of the program is just the printing of the result.



<a name="task2"></a>
## PWC 147 - Task 2

This task was tricky, not because it was conceptually complex, but because it required a lot of resources at first.
<br/>
It was asked to find out the first couple of *pentagons* numbers that summed give a pentagon number and that subtracted give also a pentagon number. A pentagon number is computed as `n * ( 3 * n - 1 ) / 2`.

<br/>
<br/>
```raku


```
<br/>
<br/>


First of all, I prepare a `%pentagons` hash that has the key as the `n` number and the value as the pentagon value.
Then I perform a nested loop other the keys of the hash to find out a couple of pentagon numbers that have the sum and the difference that provide two pentagon numbers.
Computing the `$sum` and the `$diff` is easy, and at first I thought that just `grep`ping the values would suffice. The problem is that the program never ends, I mean, after `11 minutes` it was still running.
<br/>
Therefore I introduced the `inverse-pentagons` hash that is indexed the opposite from `%pentagons`: the keys are the pentagons values and the values are the number that generated them.
So far, it just suffice to apply `:exists` and `:!exists` on such hash to see, in a very quick way, if the `$sum` and `$diff` are pentagon numbers.
This also allows me to print out a full expression that represents the result, and now the program ends in a couple of seconds:


<br/>
<br/>

``` shell
% time raku ch-2.p6
P( 1020 ) + P( 2167 ) = 1560090 + 7042750 = 8602840 = P( 2395 )
P( 1020 ) - P( 2167 ) = 1560090 + 7042750 = 5482660 = P( 1912 )
raku ch-2.p6  2,41s user 0,05s system 107% cpu 2,277 total

```





<a name="task1pg"></a>
## PWC 147 - Task 1 in PostgreSQL

A PostgreSQL implementation of the Raku version: a function to see if a number is prime and one to compute the truncated primes over a list of numbers.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
f_is_prime( n bigint )
RETURNS bool
AS
$CODE$
DECLARE
        i int;
BEGIN
        FOR i IN 2 .. ( n - 1 ) LOOP
            IF n % i = 0 THEN
               RETURN false;
            END IF;
        END LOOP;

        RETURN true;
END
$CODE$
LANGUAGE plpgsql;



CREATE OR REPLACE FUNCTION
f_generate_truncated_primes( l int = 20 )
RETURNS SETOF int
AS
$CODE$
DECLARE
        i int;
        current bigint;
        fnd   int := 0;
BEGIN
<<MAIN_LOOP>>
        FOR current IN SELECT * FROM generate_series( 10, 999999 ) LOOP
            CONTINUE WHEN current::text LIKE '%0%';

            IF NOT f_is_prime( current ) THEN
               CONTINUE MAIN_LOOP;
            END IF;


            FOR i IN 1 .. length( current::text ) LOOP
                IF NOT f_is_prime( substring( current::text FROM i )::int ) THEN
                   CONTINUE MAIN_LOOP;
                END IF;
            END LOOP;

            fnd := fnd + 1;
            RETURN NEXT current;
            IF fnd >= l THEN
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

An interesting thing to note is that the main `FOR` loop is labeled as `MAIN_LOOP` and I use quick short-circuit `CONTINUE` to start other as soon as a number is not appropriate.
<br/>
Testing if a number is a truncated prime requires to loop other the length of the number in terms of digit, `substring` moving tirght and test if the result is a prime number.


<a name="task2pg"></a>
## PWC 147 - Task 2 in PostgreSQL

I decided to implement the second task by means of generating a table, named `pentagons`, that contain the `n` number that generates the `p` pentagon. Such column is a generated (sometime called *virtual*) column.

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

Now, a function performs a record-by-record scan of the table, and for every tuple it searches for another tuple that provides the sum and difference within the table itself.
If found, a new record is returned and the function returns.

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
                     , current_tuple.p
                     , other_tuple.p
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

The `RAISE` instruction provide a descriptive output of the found solution.

<br/>
<br/>

``` sql
testdb=> select * from f_pentagons_pairs();
INFO:  P(1020) + P(2167) = P(8602840) =  1560090
INFO:  P(1020) - P(2167) = P(5482660) =  7042750
n1  |  n2  |    s    |    d    |   ps    |   pd
------+------+---------+---------+---------+---------
1020 | 2167 | 1560090 | 7042750 | 8602840 | 5482660
(1 row)

Time: 7257,715 ms (00:07,258)

```
<br/>
<br/>

As you can see, this requires a lot more time than the Raku solution, but please note I've run this on a busy server.
However, in this case SQL results in a much more compact and declarative approach than Raku is, essentially the `EXISTS` query.



<a name="task2pgb"></a>
## PWC 147 - Task 2 in PostgreSQL: a CTE only solution

The solution for the second task can be rewritten using only a *recusrive CTE*.
Instead of using a table, the content of the `pentagons` numbers can be materialized with a recursive common table expression that exploits the very same function `f_pentagon` to compute a single pentagon value.
<br/>
But most notably, instead of using a record based approach, as in the function `f_pentagon_pairs`, the query can be expressed as a full join:

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
