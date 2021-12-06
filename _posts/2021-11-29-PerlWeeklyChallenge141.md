---
layout: post
title:  "Perl Weekly Challenge 141: longer than I thought!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 141: longer than I thought!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 141](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0141/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg)





<a name="task1"></a>
## PWC 141 - Task 1
The first task was about producing a script that finds out the first ten integers that have all eight divisors.
I decided to make the script parametrizable, so it accepts the number of integers to search for and the number of divisors.


<br/>
<br/>
```raku
sub MAIN( Int $divisors = 8, Int $count = 10 ) {

    "Searching $count numbers with $divisors divisors...".say;

    my %solutions;

    for $divisors + 1  .. Inf -> $current-number {
        my @intra-solutions = ( 1 .. $current-number ).grep( { $current-number %% $_ } );

        %solutions{ $current-number } = @intra-solutions if @intra-solutions.elems == $divisors;

        last if %solutions.keys.elems >= $count;
    }

    "$_ has $divisors divisors: { %solutions{ $_ }.join( ', ' ) }".say for %solutions.keys.sort;
}

 ```
<br/>
<br/>

I loop over all the numbers, and for a specific `$current-number` I produce a list of integers that are candidates to be the divisors. Then I `grep` the list to see which one is effectively a candidate, therefore `@intra-solutions` contains all the divisors of the `$current-number`. If the array of divisors has the wanted size, I push it into the `%solutions` hash that is indexed by the `$current-number` and its divisors list.
At the end, I do print every number and the list of divisors.


<a name="task2"></a>
## PWC 141 - Task 2

This has been a huge pain to my memory: the task required to produce *live numbers*, sequences of the given input number where the digits respect the order and are all multiple of a given number.
<br/>
I did not remember the method to produce the whole range of numbers, than it came: `combinations`. That made me solve the task in a single row:


<br/>
<br/>
```raku
sub MAIN( Int $m, Int $n ) {
    $m.comb.combinations( 1 ..^ $m.Str.chars )
           .map( *.join.Int )
           .grep( * %% $n ).unique.sort.say;

}

```
<br/>
<br/>

Let's see what happens:
- `$m` is split by `comb` into its digits;
- the result is expanded into all possible `combinations`. Well, not all the combinations, rather a set limited by the length (minus one) of the length of the original number, i.e., the number of digits;
- the result is `map`ped into integers by means of `join` (e.g., `12` is seen as `1 2` and then joined);
- and the result is `grep`ped to get only multiple of `$n`;
- then I do `sort` and `unique` the result set
- and I print by means of `say`.


<a name="task1pg"></a>
## PWC 141 - Task 1 in PostgreSQL `plpgsql`

The first task is a reimplementation of its version in Raku.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
f_find_divisors( divisors int default 8, count int default 10 )
RETURNS SETOF int
AS $CODE$
DECLARE
   current_number  int;
   current_divisor int;
   current_found   int;
BEGIN
    FOR current_number IN 1 .. 999999 LOOP
        IF count = 0 THEN
           EXIT;
        END IF;

        current_found := 0;

        FOR current_divisor IN 1 .. current_number LOOP
            IF current_number % current_divisor = 0 THEN
               current_found := current_found + 1;
            END IF;
        END LOOP;

        IF current_found = divisors THEN
           count := count - 1;
           RETURN NEXT current_number;
        END IF;
    END LOOP;

    RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I start looping on a large spectre of numbers, where `current_number` assumes every single value. I then iterate on all numbers from `1` to `current_number`, that are all the candidate to be divisors: if a number is a divisor, then I increment a counter named `current_found`. At the end, if the `current_found` has the same value required, I do output the number appending it to the result set by means of `RETURN NEXT`.



<a name="task2pg"></a>
## PWC 141 - Task 2 in PostgreSQL `plpgsql`

Here I decided to use a recursive CTE to obtain all possible candidates.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
f_live_numbers( m int, n int )
RETURNS SETOF int
AS $CODE$
WITH RECURSIVE
numbers AS ( SELECT unnest( regexp_split_to_array( m::text, '' ) ) AS n )
, combinations( i, v, c ) AS (
SELECT 1, n, n
FROM numbers
UNION
SELECT i + 1, n, c || num.n
FROM combinations, numbers num
WHERE length( c || num.n ) < length( m::text ) - 1
AND num.n IN ( SELECT n FROM numbers WHERE n::int > i )
)

SELECT c::int FROM combinations
WHERE c::int % n = 0;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The `numbers` table materializes all the available digits by means of splitting the incoming number into its own set of digits.
The `combinations` table is the recursive part, and contains an index `i`, a numeric value `v` and an appended value `c`. At each iteration, a new number `v` is extracted from `numbers` and appended to the previous one. In order to guarantee ordering, a subselect checks that we only append numbers on the right of the current one from the list of numbers.
<br/>
Last, we output only those combinations where the value is a multiple of `n`.
