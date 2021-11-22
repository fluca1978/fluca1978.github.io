---
layout: post
title:  "Perl Weekly Challenge 140: bit table multiplication"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 140: bit table multiplication

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 140](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0140/){:target="_blank"}.

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
## PWC 140 - Task 1

The first task required to implement a *binary addition* so that the user provides you two bit representations, and the program has to print out the binary addition of the two.
<br/>
Thanks God, Raku has built-in functions to parse numbers from one base to another, so that the implementation could simply be:

<br/>
<br/>
```raku
sub MAIN( Int $a where { $a > 0 && $a ~~ /^ <[01]>+ $/ } ,
          Int $b where { $b > 0 && $b ~~ /^ <[01]>+ $/ } ) {

    $_.base( 2 ).say given $a.parse-base( 2 ) + $b.parse-base( 2 );
}

 ```
<br/>
<br/>

There is much more code to check the input arguments than to do the real stuff!
<br/>
However, the idea is to convert back to decimal the numbers the user has inserted, by means of `parse-base(2)`, then to sum them and convert again to binary using `base(2)`, and lastly print it out.



<a name="task2"></a>
## PWC 140 - Task 2

The second task was about a program that got three input integers: the former twos represent the boundaries of a multiplication table, the latter a value to extract from the sorted table.
<br/>
Easy enough to implement using the `X` cross join operator between lists:

<br/>
<br/>
```raku
sub MAIN( Int $i where { $i > 0 },
          Int $j where { $j > 0 },
          Int $k where { $k > 0 && $k < $i * $j } ) {

    my @table = ( 1 .. $i ) X[*] ( 1 .. $j );
    @table.sort[ $k ].say;
}

```
<br/>
<br/>

The `@table` array contains the multiplication table, done by applying `X` to two different sequences. Please note that `X[*]` does mean to apply the `*` (multiplication) between sequences.
<br/>
Then, I do `sort` the table and get the required elment to print.


<a name="task1pg"></a>
## PWC 140 - Task 1 in PostgreSQL `plpgsql`

This has been not as simple as it may sound to implement, because in PostgreSQL there are facilities to handle bit strings and masking, but not binary mathematics.
<br/>
The first *oddity* to be aware of, is that translating a string to a `varbit` (binary varying) string places the bits in the leftmost part of the string, padding with zeros the rightmost part; on the other hand, converting a string to `bit(n)` places the bits in the rightmost part of the resulting string. For this reason, I decided to limit the input of the function to be a `bit(10)` string.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION f_sum_bits( a bit(10), b bit(10) )
RETURNS text
AS $CODE$
   SELECT   ( a::int
            + b::int )::bit( 10 );

$CODE$
LANGUAGE sql;
```
<br/>
<br/>


The idea is pretty much the same as for the Raku implementation: the inputs are converted into decimal, than summed, and then converted again as a bit string.


<a name="task2pg"></a>
## PWC 140 - Task 2 in PostgreSQL `plpgsql`

This task has been implemented pretty much as in Raku, but since we don't have a `X` operator, I needed to build up the sequences of numbers as *recursive CTEs*:

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION f_multiplication_table( i int, j int, k int )
RETURNS int
AS $CODE$

WITH RECURSIVE a AS (
     SELECT 1 as x
     UNION
     SELECT x + 1 FROM a
     WHERE  x < i
)
, b AS (
    SELECT 1 as y
    UNION
    SELECT y + 1 FROM b
    WHERE  y  < j
)
, product AS ( SELECT x * y FROM a, b ORDER BY 1 )

select * from product limit 1 offset k;

$CODE$
LANGUAGE SQL;

```
<br/>
<br/>

The `a` and `b` represent a kind of materialized table with all the numbers of the sequence, then `product` is the cross join of the previous two already ordered by the values.
<br/>
Then I use two PostgreSQL keywords to get to the result:
- `limit` means how many tuples I want in the result set, in this case only one;
- `offset` means at which tuple I want the result set to start.

Therefore, `limit 1 offset k` means to pick only the `k`-nth element of the result set.
