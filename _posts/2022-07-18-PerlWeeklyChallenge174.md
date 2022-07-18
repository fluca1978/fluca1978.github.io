---
layout: post
title:  "Perl Weekly Challenge 174: the power of permutations"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 174: the power of permutations

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 174](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0174/){:target="_blank"}.

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
## PWC 174 - Task 1

Produce *Disarium* numbers, those that have the sum of the power of their digits equal to the number itself.
Seems a task for a `lazy gather` kind of approach:



<br/>
<br/>
```raku
sub MAIN( Int $limit = 19 ) {

    my @disarium-numbers = lazy gather {
        for 10 .. Inf {
            my $index = 0;
            take $_ if $_.comb.map( * ** ++$index ).sum == $_;
        }
    };

    @disarium-numbers[ 0 .. $limit ].join( "\n" ).say;
}

 ```
<br/>
<br/>


The problem is that on my poor little computer, getting more than 10 numbers requires too much time!
Probably this can become faster with some math trick and properties of these set of numbers that I have no time to dig.

<a name="task2"></a>
## PWC 174 - Task 2

Given an input array, assuming it does not contains duplicates, seek the position of such array in all possible permutations and the permutation at a given offset.

<br/>
<br/>
```raku
sub permutation2rank( @a ) {
    my $index = 0;
    my @sorted = @a.permutations.sort;
    for @sorted {
        return $index if $_ ~~ @a;
        $index++;
    }
}

sub rank2permutation( @a, $offset ) {
    return @a.permutations.sort[ $offset ];
}


sub MAIN( *@input where { @input.grep( * ~~ Int ).elems == @input.elems } ) {

    say permutation2rank( @input);
    say rank2permutation( @input, 1 );
}
```
<br/>
<br/>

The `permutation2rank` function takes the input array, computes all the `permutations` and then `sort`s them. Then it look where the input array is within this list of permutations.
On the other hand, the `rank2permutation` takes in input the array and the rank positiopn, and provides the corect permutation.
<br/>
The trick here is that the `permutations` and `sort` methods can be applied together.

<a name="task1plperl"></a>
## PWC 174 - Task 1 in PostgreSQL PL/Perl

Very likely the Raku solution:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc174.task1_plperl( int )
RETURNS SETOF BIGINT
AS $CODE$

my ( $limit ) = @_;
my ( $current ) = 9;
while ( $limit > 0 ) {
     $current++;
     my $index = 0;
     my @digits = map { $_ ** ++$index } split( //, $current );
     my $sum = 0;
     $sum += $_  for ( @digits );

     if ( $current == $sum ) {
        $limit--;
        return_next( $current );
     }
}

return undef;

$CODE$
LANGUAGE plperl;
```
<br/>
<br/>

Note that usage of `bigint`s, that can overflow when numbers start growing.


<a name="task2plperl"></a>
## PWC 174 - Task 2 in PostgreSQL PL/Perl

Similar to the Raku solution, but here we cannot just `sort` an array, so the trick is to translate the array into a string, sort the string, and re-split the array when found.


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc174.task2_permutation2rank( int[] )
RETURNS int
AS $CODE$

use List::Permutor;

my $input = shift;
elog( DEBUG, "INPUT " . join( ",", @{ $input } ) );

my @permutations;
my $permutator   = List::Permutor->new( @{ $input } );
while ( my @current = $permutator->next ) {
      push @permutations, join( '', @current );
}

@permutations = sort @permutations;

for ( 0 .. $#permutations ) {
    return $_ if $permutations[ $_ ] == join( '', @{ $input } );
}

return -1;
$CODE$
LANGUAGE plperlu;


CREATE OR REPLACE FUNCTION
pwc174.task2_rank2permutation( int, int[] )
RETURNS int[]
AS $CODE$

use List::Permutor;

my $index = shift;
my $input = shift;
elog( DEBUG, "INPUT " . join( ",", @{ $input } ) );

my @permutations;
my $permutator   = List::Permutor->new( @{ $input } );
while ( my @current = $permutator->next ) {
      push @permutations, join( '-', @current );
}

@permutations = sort @permutations;
return [ split '-', @permutations[ $index ] ];

return undef;
$CODE$
LANGUAGE plperlu;
```
<br/>
<br/>

Note the usage of `plperlu` as language, since I used an external module to quickly get the list of all possible permutations.
Also note the usage of an array translated from SQL to Perl for the `$input` container.


<a name="task1plpgsql"></a>
## PWC 174 - Task 1 in PostgreSQL PL/PgSQL

Same approach as in Raku:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc174.task1_plpgsql( l int DEFAULT 19 )
RETURNS SETOF BIGINT
AS $CODE$
DECLARE
        i int;
        v bigint;
        n bigint;
        s bigint;
BEGIN
        FOR n IN 10 .. 999999 LOOP
            i := 1;
            s := 0;

            FOR v IN SELECT * FROM regexp_split_to_table( n::text, '' ) LOOP
                s := s + pow( v::bigint, i );
                i := i + 1;
            END LOOP;

            IF s = n THEN
               l := l - 1;
               RETURN NEXT n;
            END IF;

            IF l <= 0 THEN
               EXIT;
            END IF;
        END LOOP;
RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Here there is the need to use `pow` to compute the power of `v`, and the `regexp_split_to_table` function requires some conversions from `text` to `int`, but the workflow is the same already seen in the other implementations.

<br/>
There is also a single query approach:

<br/>
<br/>

``` sql
WITH digits AS
(
   SELECT v, digits.*, pow( digits.d, digits.rn) AS p
   FROM generate_series( 10, 99999 ) v
   , LATERAL ( SELECT d::bigint, row_number() over () AS rn
       FROM regexp_split_to_table( v::text, '') d
     ) digits

)
, comparison AS
(
   SELECT v, sum( p ) as s
   FROM digits
   GROUP BY v
)
SELECT *
FROM comparison
WHERE v = s
ORDER BY v
;

```
<br/>
<br/>

The idea is that `digits` will combine a number from `generate_series` with its digits and the position of such digit, then `comparison` will compute the sum of the power and the outer query will filter only those rows that have a match.

<a name="task2plpgsql"></a>
## PWC 174 - Task 2 in PostgreSQL PL/PgSQL

Since permutating arrays in PL/PgSQL is not that fun, I used a trick here: I call the PL/Perl functions!
I thinks this is useful, because it demonstrates how to integrate Perl and SQL together.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc174.task2_plpgsql_permutation2rank( input int[] )
RETURNS int
AS $CODE$
BEGIN
        RETURN pwc174.task2_permutation2rank( input );
END
$CODE$
LANGUAGE plpgsql;



CREATE OR REPLACE FUNCTION
pwc174.task2_plpgsql_rank2permutation( i int, input int[] )
RETURNS int[]
AS $CODE$
BEGIN
RETURN pwc174.task2_rank2permutation( i, input );
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>
