---
layout: post
title:  "Perl Weekly Challenge 144: headache!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 144: headache!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 144](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0144/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 2 in plpgsql](#task2pg)





<a name="task1"></a>
## PWC 144 - Task 1
The first task was about finding out all *semiprimes* numbers lower than `100`. A semiprime number is a number obtained by multiplying two prime numbers.
<br/>
My first attempt was by doing a nested loop, than I modified it to be a single loop:


<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 1 } = 100 ) {
    my @semi-primes;

    for 1 .. $limit -> $current-number {
        @semi-primes.push: $current-number if ( 1 .. $current-number ).grep( { $_.is-prime
                                                                               && $current-number %% $_
                                                                               && ( $current-number / $_ ).Int.is-prime } )
                            .elems > 0;
    }

    @semi-primes.join( ',' ).say;
}
 ```
<br/>
<br/>

Given a `$limit`, in this case `100`, I do iterate on all integers up to such limit. For every number, I keep it as semiprime if, given all factors, I can find a couple of factors that are primes. To find out such couple, I look for all primes lower than the specific number and that, if I divide the number by such prime, give me another prime.



<a name="task2"></a>
## PWC 144 - Task 2

The second task was complicated, at least according to me: it was related t producing an Ulmar sequence.
The problem was *to understand* how to implement such Ulamr sequence, because it was not really clear to me.
However, according to the results, it seems it can be implemented in two steps:
- provide a way to sum the Ulmar sequence built so far, finding out the unique sums (this was hard for me to understand from the description);
- add the lowest number in the sum to the sequence.

<br/>
Let's start from the first step: the `do-sum` function accepts an array (the Ulmar sequence itself) and returns the unique sums of elements from left to right. I use an ugly hash vivification to find out all the unique sums.

<br/>
<br/>
```raku
sub do-sum( @array ) {
    my %sums;

    for 0 .. @array.end -> $index-left {
        for $index-left ^.. @array.end -> $index-right {
            next if $index-left == $index-right;

            %sums{ @array[ $index-left ] + @array[ $index-right ] }++;
        }
    }


    my @found;
    @found.push: $_.Int if %sums{ $_ } == 1 for %sums.keys;
    @found;

}

```
<br/>
<br/>


Then there is the `MAIN`, that iterates to get a new value for the sequence:

<br/>
<br/>

``` raku
sub MAIN( Int $u where { $u > 0 },
          Int $v where { $v > 0 },
          Int $limit = 10 ) {

    my @ulam = $u, $v, $u + $v;

    while @ulam.elems < $limit {
        @ulam.push: do-sum( @ulam ).grep( { $_ > @ulam[ * - 1 ] } ).min;
    }

    @ulam.join( ',' ).say;

}
```
<br/>
<br/>

The loop adds a new element to the `@ulmar` array only taking into account the `min` value of the susms that are greater than the last element of the array itself.


<a name="task1pg"></a>
## PWC 144 - Task 1 in PostgreSQL

The first task is a line-by-line implementation of the Raku solution, but since there are no such base functions in PostgreSQL, I did create some of them to make the code look nicer.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION f_is_prime( n int )
RETURNS bool
AS $CODE$
DECLARE
        divisor int;
BEGIN
        FOR divisor IN 2 .. ( n - 1 ) LOOP
            IF mod( n, divisor ) = 0 THEN
               RETURN false;
            END IF;
        END LOOP;

        RETURN TRUE;
END
$CODE$
LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION f_semiprime_factors( n int )
RETURNS SETOF int
AS $CODE$
DECLARE
        current_number int;
BEGIN
        FOR current_number IN 1 .. n LOOP
            IF f_is_prime( current_number ) AND mod( n, current_number ) = 0 AND f_is_prime(  ( n / current_number ) ) THEN
               RETURN NEXT current_number;
            END IF;
        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;



CREATE OR REPLACE FUNCTION f_is_semiprime( n int )
RETURNS bool
AS $CODE$
   SELECT CASE count( * )
          WHEN 0 THEN false
          ELSE true
          END
          FROM f_semiprime_factors( n ) s;
$CODE$
LANGUAGE sql;



CREATE OR REPLACE FUNCTION f_find_semiprimes( lim int default 100 )
RETURNS SETOF int
AS $CODE$
DECLARE
      n int;
BEGIN
   n := 1;

   WHILE n < lim LOOP
       IF f_is_semiprime( n ) THEN
          RETURN NEXT n;
       END IF;
       n := n + 1;
   END LOOP;

   RETURN;
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>

The `f_is_prime` function provides `true` or `false` depending if its input number is a prime.
The `f_semiprime_factors` gives a result set of integers that represents all the semiprimes factors of a given number. Then, the `f_semiprime` function iterates over all integers up to the given limit `lim` and returns a result set of integers that represent only the semiprime condition.
<br/>
<br/>
Having `f_is_semiprime` in place, it is also possible to issue another query, without the need for `f_semiprime`:


<br/>
<br/>

``` sql
testdb=> select v FROM generate_series( 1, 100 ) v WHERE f_is_semiprime( v );
```
<br/>
<br/>

The trick here is to use `generate_series` to generate all numbers and filter by `f_is_semiprime`.

<a name="task2pg"></a>
## PWC 144 - Task 2 in PostgreSQL

Producing the Ulmar sequence in PostgreSQL was simpler than in Raku this time, at least because I have already clear how to implement it. I decided to provide two functions, to split the task in a similar way to its Raku implementation:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION f_ulam_do_sum( ulam int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
        left_index int;
        right_index int;
BEGIN

        FOR left_index IN 1 .. array_length( ulam, 1 ) LOOP
            FOR right_index IN left_index + 1 .. array_length( ulam, 1 ) LOOP
                RETURN NEXT ulam[ left_index ] + ulam[ right_index ];
            END LOOP;
        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION f_ulam( u int, v int, lim int default 10 )
RETURNS int[]
AS $CODE$
DECLARE
        ulam int[];
        next_value int;
BEGIN

        ulam := ulam || u || v || u + v;

        WHILE array_length( ulam, 1 ) < lim LOOP

              SELECT vv
              INTO next_value
              FROM f_ulam_do_sum( ulam ) AS sums( vv )
              WHERE vv > ulam[ array_length( ulam, 1 ) ]
              GROUP BY 1
              HAVING COUNT( * ) = 1
              ORDER BY vv
              LIMIT 1;

              ulam := ulam || next_value;
        END LOOP;

        RETURN ulam;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The `f_ulam_do_sums` function accepts an array of integers, the Ulam sequence built so far, and produces an array of sums.
The `f_ulam` starts with the two initial values, builds an array with the first three elements, and then performs a query that produces the very next element to append to the array.
I use the array concation `||` to extend the array.
The SQL query can be split into parts:
- it asks the `f_ulam_do_sums` to provide the sums of the sequence so far;
- it orders the list of sums so that the smallest one is the first;
- it limits the query results to only one entry, thus the *min* value;
- groups the results by value of sums, taking only those that have a count of `1` (i.e., appear only once);
- selects only values that are greater than the last element of the current Ulam array.
