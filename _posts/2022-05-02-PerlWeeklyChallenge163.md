---
layout: post
title:  "Perl Weekly Challenge 163: the infinite loop!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 163: the infinite loop

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 163](https://perlweeklychallenge.org/blog/perl-weekly-challenge-163/){:target="_blank"}.

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
## PWC 163 - Task 1

This task was about doing an integer sum of bitwise ands within an array of numbers. It was not complicated, but it was requiring a full coverage of the index, that means a last step of using the very first and very last elements was required, as well as some conversions from and to integers:



<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems } ) {
    my $sum = 0;
    $sum += ( @n[ $_ - 1 ].base( 2 ) +& @n[ $_ ].base( 2 ) ).Str.parse-base( 2 ) for 1 ..^ @n.elems;
    $sum += ( @n[ 0 ].base( 2 ) +& @n[ * - 1 ].base( 2 ) ).Str.parse-base( 2 );
    $sum.say;
}

 ```
<br/>
<br/>


<a name="task2"></a>
## PWC 163 - Task 2

This task was starting from an array of integers, that have to be progressively reduced by creating a new array with a sum of elements between the previous array and the previous element of the current array.

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems } ) {

    my @matrix;
    @matrix.push: [ @n ];

    while ( @matrix[ * - 1 ].elems > 2 ) {
        my @row;
        @row.push: @matrix[ * - 1 ][ 1 ] ;

        for 2 ..^ @matrix[ * - 1 ].elems {
            @row.push: @matrix[ * - 1 ][ $_ ] + @row[ * - 1 ];
        }

        @matrix.push: [ @row ];
    }

    @matrix.push: [ @matrix[ * - 1 ][ * - 1 ] ];

    say @matrix[ * - 1 ][ 0 ];
}

```
<br/>
<br/>

At every step, a new `@row` array of the `@matrix` is built, and every element of the current `@row` is made by the second element of the previous row in the matrix, then the sum of the same column element of the current row with the previous element of the current row.
Once the matrix has reached a single element row, the job is done.

## PWC 163 - Task 1 in PostgreSQL PL/Perl


A quite simple translation of the Raku implementation:

<br/>
<br/>

``` perl
CREATE OR REPLACE FUNCTION
pwc163.task1_plperl( int[] )
RETURNS int
AS $CODE$
my ( $n ) = @_;

my $sum = 0;
for my $index ( 1 .. scalar( @$n ) - 1 ) {
    my ( $a, $b ) = ( sprintf( '%03b', $n->[ $index - 1 ] ), sprintf( '%03b', $n->[ $index ] ) );
    my $result = $a & $b;

    elog( DEBUG, "Nums $n->[ $index - 1 ] and $n->[ $index ] => $a and $b => $result" );

    $sum += oct( '0b'. $result );
}

my ( $a, $b ) = ( sprintf( '%03b', $n->[ 0 ] ), sprintf( '%03b', $n->[ -1 ] ) );
my $result = $a & $b;
$sum += oct( '0b'. $result );

return $sum;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Note that I use `sprintf` and `oct` as ways to implement Raku `base` and `parse-base` methods. Despite this, the alghoritm is the same.

<a name="task2plperl"></a>
## PWC 163 - Task 2 in PostgreSQL Pl/Perl

Same Raku implementation, but with a trick that made me run into *an infinite loop*:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc163.task2_plperl( int[] )
RETURNS int
AS $CODE$

my ( $n ) = @_;
my @matrix;

push @matrix, [ @$n ];
 while ( scalar( @{ $matrix[ -1 ] } ) > 2 ) {
      my @row;
      push @row, $matrix[ -1 ][ 1 ];
      for my $index ( 2 .. scalar( @{ $matrix[ -1 ] } ) - 1 ) {
          push @row,  $matrix[ -1 ][ $index ] + $row[ -1 ];
      }

      elog( DEBUG, "Current row: " . join( ',', @row ) );
      push @matrix, [ @row ];
 }

return $matrix[ -1 ][ -1 ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

At first, I wrongly forgot that in Pl/Perl **arrays are not really Perl arrays**: they are an *iterable object* that smells like a Perl array, but needs to be derefenced pretty much any time. So when I looped up to `scalar $matrix[ -1 ]` I was issuing an infinite loop because I needed to deference the object asn an array!


## PWC 163 - Task 1 in PostgreSQL Pl/PgSQL

Quite simple, since it is possible to cast a number to `bit` and then to `int` again:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc163.task1_plpgsql( n int[] )
RETURNS int
AS $CODE$
DECLARE
        summy int := 0;
        index int;
BEGIN
       FOR index IN 2 .. array_length( n, 1 )  LOOP
           summy := summy
                    + ( n[ index - 1 ]::bit(8)
                        &
                        n[ index ]::bit( 8 ) )::int;
       END LOOP;

       summy := summy
                + ( n[ 1 ]::bit(8)
                    &
                    n[ array_length( n, 1 ) ]::bit( 8 ) )::int;

      RETURN summy;
END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>


<a name="task2plpgsql"></a>
## PWC 163 - Task 2 in PostgreSQL Pl/PgSQL

This time I took a slightly different approach than those in Raku and PL/Perl: I split the alghoritm in two steps. The first one was a function able to reduce an incoming array into the one with its sums, essentially producing from one row the next one. The second function was a simple loop over the previous one until the array got a single dimension.
<br/>
**Take care that in SQL arrays start from `1`!**

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc163.task2_reduce( n int[] )
RETURNS int[]
AS $CODE$
DECLARE
        summy int;
        index int;
        res   int[];
BEGIN

        FOR index IN 2 .. array_length( n, 1 ) LOOP
            IF index = 2 THEN
               res := array_append( res, n[ index ] );
            ELSE
               res := array_append( res,  n[ index ] + res[ array_length( res, 1 ) ] );
            END IF;
        END LOOP;

        RETURN res;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc163.task2_plpgsql( n int[] )
RETURNS int
AS $CODE$
DECLARE
BEGIN
        WHILE array_length( n, 1 ) > 1 LOOP
              n := pwc163.task2_reduce( n );
        END LOOP;

        RETURN n[ 1 ];
END
$CODE$
LANGUAGE plpgsql;

```
