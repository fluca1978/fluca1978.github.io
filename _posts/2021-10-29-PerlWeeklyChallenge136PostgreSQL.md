---
layout: post
title:  "Perl Weekly Challenge 136: PostgreSQL Solutions"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge, this time in PostgreSQL!

# Perl Weekly Challenge 136: PostgreSQL Solutions

Wait a minute, what the hell is going on? A Perl challenge and PostgreSQL?
<br/>
Well, it is almost two years now since I've started participating regurarly in the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"}, and I always solve the tasks in Raku (aka Perl 6).
<br/>
Today I decided to spend a few minutes in order to try to solve the assigned tasks in PostgreSQL. And I tried to solve them in an SQL way: declaratively.
<br/>
<br/>
So here there are my solutions in PostgreSQL for the [Challenge 136](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0136/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)




<a name="task1"></a>
## PWC 136 - Task 1

The first task asked to find out if two numbers are friends, meaning that their *greatest common divisor* should be a positive power of `2`. This is quite easy to implement in pure SQL:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION friendly( m int, n int )
RETURNS int
AS $CODE$
   SELECT
        CASE gcd( m, n ) % 2
             WHEN 0  THEN 1
             ELSE 0
        END;
$CODE$
LANGUAGE SQL;

```
<br/>
<br/>
The `gcd` function finds out the greatest common divisor, then I apply the module `%` operator and catch the remainder: if it is `0` then the gcd is a power of `2`, else it is not.

<a name="task2"></a>
## PWC 136 - Task 2

The second task was much more complicated to solve, and required, at least to me, a little *try-and-modify* approach. Given a specific value, we need to find out all unique combinations of numbers within the Fibonacci sequence that can lead to that value sum.
<br/>
I decided to solve it via a `RECURSIVE` Common Table Expression (CTE), due to the fact I need to produce a Fibonacci series:

<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION fibonacci_sum( l int DEFAULT 16 )
RETURNS bigint
AS $CODE$

WITH RECURSIVE
fibonacci( n, p ) AS
(
        SELECT 1, 1
        UNION
        SELECT p + n, n
        FROM fibonacci
        WHERE n < l
)
, permutations AS
(
        SELECT n::text AS current_value, n as total_sum
        FROM fibonacci
        UNION
        SELECT current_value || ',' || n, total_sum + n
        FROM permutations, fibonacci
        WHERE
                position( n::text in  current_value ) = 0
       AND n > ALL( string_to_array( current_value, ',' )::int[] )


)
SELECT count(*)
FROM permutations
WHERE total_sum = l
;

$CODE$
LANGUAGE SQL;

```
<br/>
<br/>

The searched value is the argument to the function, that is `l`.
<br/>
The first part of the CTE computes the Fibonacci sequence of values that lead to `l`, and thus we can throw away all the other values since their sum will be greater than `l`.
<br/>
The `permutations` CTE computes a two column materialization: each value from the Fibonacci sequence is appended to the next value, and the sum so far is computed. Note the `WHERE` clause:
- the `position` function checks that the digit has not already be inserted in the list;
- the `n > ALL` considers only ordered values, that is `3,5` is a good list, but `5,3` is not because `n` is `5`.

Thanks to the trick of considering only ordered sequences, I can trim out all the sequences that produce the same sum, with the same numbers, in a different order. For example `3, 13` and `13,3` produce the same value, but only the first one is kept.
<br/>
At this point, it does suffice to count how many tuples there are in `permutations` to get final answer of the task: how many permutations that lead to `l` by sum can be found in the Fibonacci series.


# Conclusions

Clearly PostgreSQL provides all the features to implement *program-like* behaviors in a declarative way. Of course, the above solutions are neither the best nor the more efficient that can be implemented, but they demonstrate how powerful PostgreSQL (and more in general, SQL), can be to solve tasks where a few nested loops seem the simpler approach!
