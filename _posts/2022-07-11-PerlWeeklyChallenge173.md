---
layout: post
title:  "Perl Weekly Challenge 173: sly"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 173: sly

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 173](https://perlweeklychallenge.org/blog/perl-weekly-challenge-173/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)


Last, the solutions in PostgreSQL PL/PgSQL:

<br/>
- [Task 1 in PostgreSQL Pl/PgSQL](#task1plpgsql)
- [Task 2 in PostgreSQL Pl/PgSQL](#task2plpgsql)

<a name="task1"></a>
## PWC 173 - Task 1

Decide if a given number is esthetic, that means that every digit from left to right is different from the previous one by one unit:




<br/>
<br/>
```raku
sub MAIN( Int $n where { $n >= 10 } ) {
    my @digits = $n.comb;
    for 0 ..^ @digits.elems - 1 -> $index {
        "Not esthetic $n".say and exit if abs( @digits[ $index ] - @digits[ $index + 1 ] ) != 1
    }

    "$n is esthetic!".say;
}
 ```
<br/>
<br/>


The idea is simple: iterate on all possible digits and see if the current one and the rightmost one are separated by one unit. Break the program as soon as this condition is not met.


<a name="task2"></a>
## PWC 173 - Task 2

Compute the Sylverster's sequence, hence the name of the post to *Sly*.
But, it's not the same Sylvester!
<br/>
The sequence is quite simple to compute: bootstraps with `2` and `3`, and the continues with the product of all preceeding terms added by one:

<br/>
<br/>
```raku
sub MAIN( Int $limit = 10 ) {
    my @sly = 2, 3;

    while ( @sly.elems < $limit ) {
        @sly.push: ( [*] @sly ) + 1;
    }

    @sly.join( "\n" ).say;
}

```
<br/>
<br/>

Here I use the `[*]` reduction operator to quickly compute the product.

<a name="task1plperl"></a>
## PWC 173 - Task 1 in PostgreSQL PL/Perl

Very likely the Raku solution:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc173.task1_plperl( int )
RETURNS BOOL
AS $CODE$
my ( $n ) = @_;
return 0 if $n < 10;

my @digits = split //, $n;

for ( 0 .. $#digits - 1 ) {
    return 0 if ( abs( $digits[ $_ ] - $digits[ $_ + 1 ] ) != 1 );
}

return 1;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 173 - Task 2 in PostgreSQL PL/Perl

Same as the Raku solution:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc173.task2_plperl( int )
RETURNS SETOF BIGINT
AS $CODE$

my ( $limit ) = @_;

my @sly = ( 2, 3 );

return_next( $sly[ 0 ] );
return_next( $sly[ 1 ] );

while ( $#sly < $limit ) {
      my $new_value = 1;
      $new_value *= $_ for (@sly);
      push @sly, ++$new_value;
      return_next( $new_value );
}

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

One possible problem is that `bigint` is going out of range, with Perl using the scientific notation that is not understood by `bigint`.


<a name="task1plpgsql"></a>
## PWC 173 - Task 1 in PostgreSQL PL/PgSQL

There is a function to do the job:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc173.task1_plpgsql( n int )
RETURNS BOOL
AS $CODE$
DECLARE
        d text;
        previous int := -1;
BEGIN
        FOR d IN SELECT regexp_split_to_table( n::text, '' ) LOOP
            IF previous = -1 THEN
               previous = d::int;
               CONTINUE;
            END IF;

            IF abs( previous - d::int ) != 1 THEN
               RETURN FALSE;
            ELSE
                previous = d::int;
            END IF;
        END LOOP;

RETURN TRUE;
END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>

The function keeps track of the `previous`ly seen number and exists with a false value as soon as the difference between the current one and the previous one is not `1`.
<br/>
There is also a single query approach:

<br/>
<br/>

``` sql
WITH nums AS
(
        SELECT v::int, v::int - lag( v::int, 1, 0 ) OVER () AS diff
        FROM regexp_split_to_table( 12345::text, '' ) v
)
SELECT NOT EXISTS(
       SELECT *
       FROM nums
       WHERE abs( diff ) <> 1
);

```
<br/>
<br/>

The CTE takes the number and splits it into a table, where `lag` is used to compute the difference between the number and the previous one (i.e., the one of the previous row).
The outer query selects rows where the difference is not of one unit, and in the case such rows exists, the query returns a false value to indicate that the number is not esthetic.


<a name="task2plpgsql"></a>
## PWC 173 - Task 2 in PostgreSQL PL/PgSQL

Since there is no aggregation for multiplication, I implemented a simple function:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc173.task2_plpgsql( n int default 10 )
RETURNS SETOF BIGINT
AS $CODE$
DECLARE
        product bigint;
BEGIN

        product := 2 * 3;
        RETURN NEXT 2;
        RETURN NEXT 3;
        n := n - 2;

        WHILE n > 0 LOOP
              RETURN NEXT ( product + 1 );
              product := ( product + 1 ) * product;
              n := n - 1;
        END LOOP;

RETURN;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The function does return the bootstrap terms, and then keeps track of the previously computed product, so that it can quickly recompute the next one. As already stated, there could be an overflow with bigintegers and long sequences.

Even here it is possible to solve the problem with a single query:


<br/>
<br/>

``` sql
WITH RECURSIVE numbers AS
(
        SELECT 2::numeric AS v, 0::numeric AS p, 1 AS r
        UNION
        SELECT 3 AS v, 6 as p, 2 AS r

        UNION

        SELECT p + 1, (p + 1) * p, r + 1
        FROM numbers
        WHERE p <> 0

)
SELECT v
FROM numbers
LIMIT 10;

```
<BR/>
<BR/>

The query provides the bootstrap terms `2` and `3`, as well as a precomputed product `p`. Then the recursive term computes the next value as the previous product `p` added by one, and prepares the next product.
The outer query does imposes the limit to avoid PostgreSQL to compute the whole universe!
