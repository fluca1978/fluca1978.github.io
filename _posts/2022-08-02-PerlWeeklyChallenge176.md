---
layout: post
title:  "Perl Weekly Challenge 176: scrumbled numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 176: scrumbled numbers

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 176](https://perlweeklychallenge.org/blog/perl-weekly-challenge-176/){:target="_blank"}.

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
## PWC 176 - Task 1

Find the first and lowest integer number so that the equivalent numbers multiplied by a factor ranging from 2 to 6 are composed by the very same digits. The idea is quite simple:
- grab a number;
- compute the multiplied `@values`;
- compute the `@permutations` of the original number and make them integers;
- see if the number of `$found` permutations that are in the `@values` multiplied is the right number.



<br/>
<br/>
```raku
sub MAIN() {

    for 1 .. Inf -> $current {
        my @values = (1 .. 6).map: { $current * $_ };
        # compute the permutations of the current number
        my @permutations = $current.comb.permutations.map: *.join.Int;

        my $found = 0;
        for @values {
            $found++ if $_ == any(@permutations);
        }

        $current.say and last if $found == @values.elems;
    }
}

 ```
<br/>
<br/>



<a name="task2"></a>
## PWC 176 - Task 2

Find out all *reversable* numbers up to `100`: a number that summed with its mirrored version provides a value made only by odd digits. Quite simple to implement:

<br/>
<br/>
```raku
sub MAIN( Int $limit = 100 ) {

    for 1 .. $limit {
        $_.say if ! ( $_ + $_.flip.Int ).comb.grep( * %% 2 );
    }
}

```
<br/>
<br/>

But my first implementation was wrong: I used `reverse` instead of `flip`, so I got more numbers than those I was expected to get!

<a name="task1plperl"></a>
## PWC 176 - Task 1 in PostgreSQL PL/Perl

The same Raku solution, with a little more work due to chaining `split` (to get the digits), `sort` and `join`:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc176.task1_plperl()
RETURNS BIGINT
AS $CODE$

for my $current ( 1 .. 999999 ) {

    my @values = map { $_ * $current } ( 2 .. 6 );

    my $found = 1;
    for ( @values ) {
        my $sorted_digits = join( '', sort( split( '', $current ) ) );
        my $other_digits  = join( '', sort( split( '', $_ ) ) );

        $found = 0 if ( $sorted_digits != $other_digits );
        last if ! $found;
    }

    next if ! $found;
    return $current;
}

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The idea is the same: I do `map` the original number multiplying it by a factor, then use `split` to get the digits, `sort` to place them in the same order, and `join` to make it a number again. If two numbers undergoing this chain of methods result in the same value, it means they have the same digits!


<a name="task2plperl"></a>
## PWC 176 - Task 2 in PostgreSQL PL/Perl


The idea is similar to the Raku implementation, with some extra work due to method chaining:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc176.task2_plperl( int )
RETURNS SETOF INT
AS $CODE$

my ( $limit ) = @_;
$limit //= 100;

for my $current ( 1 .. $limit ) {
    my $sum = $current + reverse( $current );
    my @digits = split( '', $sum );
    return_next( $current ) if ( ! grep( { $_ % 2 == 0 } @digits ) );
}

return undef;


$CODE$

```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 176 - Task 1 in PostgreSQL PL/PgSQL

In the beginning I thought about simply calling the PL/Perl implementation, than I implemented it with a pure PL/PgSQL function:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc176.task1_plpgsql()
RETURNS BIGINT
AS $CODE$
DECLARE
        i int;
        m int;
        v int;
        c int;
BEGIN
<<main_loop>>
     FOR i IN 1 .. 999999 LOOP
         FOR m IN 2 .. 6 LOOP
             v := m * i;
             c := 0;

             SELECT count( vv )
             INTO c
             FROM regexp_split_to_table( v::text, '' ) vv
             WHERE vv NOT IN ( SELECT * FROM regexp_split_to_table( i::text, '') );

             CONTINUE main_loop WHEN c <> 0;
         END LOOP;

         -- if here, ok!
         RETURN i;
     END LOOP;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

For any given number `i` I do compute its multiplication by a factor `m` and store it into `v`. Then I execute a query to see if all digits (via `regexp_split_to_table()`) of `v` are within the digits of `i`: I do count and place into `c` the number of digits that do not correspond. Therefore, if `c` is not zero, the current number has a multiple that has not the same digits, so I can restart (or continue) the loop with a next `i`.


<a name="task2plpgsql"></a>
## PWC 176 - Task 2 in PostgreSQL PL/PgSQL

Simple enough to be implemented with a check done by a query: for a given number `i` I do compute the sum with its mirrored part and count how many even digits are there in the resulting sum. If the count is not zero, the number is not reversable, otherwise I can return it within the result set.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc176.task2_plpgsql( l int default 100 )
RETURNS SETOF INT
AS $CODE$
DECLARE
        i int;
        s int;
        c int;
BEGIN

        FOR i IN 1 .. l LOOP
            s := i + reverse( i::text )::int;
            c := 0;

            SELECT count( v )
            INTO c
            FROM regexp_split_to_table( s::text, '' ) v
            WHERE v::int % 2 = 0;

            IF c = 0 THEN
               RETURN NEXT i;
            END IF;
        END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
