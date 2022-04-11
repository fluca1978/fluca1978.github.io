---
layout: post
title:  "Perl Weekly Challenge 160: English equilibirum"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 160: English equilibrium

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 160](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0160/){:target="_blank"}.

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
## PWC 160 - Task 1

This task was harder to understand than to implement: given an integer number between `1` and `9`, find out a chain that leads to `4`, writing the English word representing every number. At every step, the next number is the length of the English word of the current step.
I used an hash to the aim:




<br/>
<br/>
```raku
sub MAIN( Int :$n! where { 0 < $n < 10 }, Int :$stop = 4 ) {
    my %words = 1 => 'One',
                2 => 'Two',
                3 => 'Three',
                4 => 'Four',
                5 => 'Five',
                6 => 'Six',
                7 => 'Seven',
                8 => 'Eight',
                9 => 'Nine';


    my $current = $n;
    while ( $current != $stop ) {
        "{ %words{ $current } } is { %words{ $current }.chars }".say and $current = %words{ $current }.chars;
    }

    "%words{ $stop } is magic".say;

}

 ```
<br/>
<br/>

The `$stop` criteria is hitting `4`, and unless `$current` has such value I keep going on extracting the value and printing out the sentence and going to the next value.

<a name="task2"></a>
## PWC 160 - Task 2

The Equilibrium index of an array is such an index that makes the array split into two parts so that the sum of both parts is the same. Quite easy to implement:

<br/>
<br/>
```raku
sub MAIN( *@A where { @A.grep( * ~~ Int ).elems == @A.elems } ) {
    for 1 ..^ @A.elems {
        .say and exit if @A[ 0 .. $_ - 1 ].sum == @A[ $_ .. * - 1 ].sum;
    }
    '-1'.say;

}
```
<br/>
<br/>

I use array slicing and the `sum` function to test if I hit the equilibrium index.


<a name="task1plperl"></a>
## PWC 160 - Task 1 in PostgreSQL PL/Perl


Pretty much the same implementation done in Raku:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc160.four_is_magic( int )
RETURNS SETOF text
AS $CODE$

my $words = {
   1 => 'One',
   2 => 'Two',
   3 => 'Three',
   4 => 'Four',
   5 => 'Five',
   6 => 'Six',
   7 => 'Seven',
   8 => 'Eight',
   9 => 'Nine',
};


my $stop = 4;
my ( $current ) = @_;

while ( $current != $stop ) {
      my $word = $words->{ $current };
      my $size = length $word;
      $current = $size;

      return_next( "$word is $size" );
}

return_next( $words->{ $stop } . " is magic" );
return undef;
$CODE$
LANGUAGE plperl;
```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 160 - Task 2 in PostgreSQL Pl/Perl

Same implementation of the Raku one, with an anonymous function used to compute the sum:

<br/>
<br/>

``` perl
CREATE OR REPLACE FUNCTION
pwc160.equilibrium( int[] )
RETURNS int
AS $CODE$
my ( @A ) = @{ $_[0] };

# compute the sum of an array
my $do_sum = sub {
   my ( @a ) = @_;
   my $sum = 0;
   $sum += $_  for ( @a );
   return $sum;
};

for my $index ( 1 .. $#A ) {
    return $index if ( $do_sum->( @A[ 0 .. $index - 1 ] ) == $do_sum->( @A[ $index .. $#A ] ) );
}

return -1;
$CODE$
LANGUAGE plperl;
```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 160 - Task 1 in PostgreSQL Pl/PgSQL

I decided to use a table to keep track of the English words for every number, as well as a *generated column* that contains the length of every word.

<br/>
<br/>

``` sql
CREATE TABLE IF NOT EXISTS pwc160.words
(
   pk int generated always as identity
   , n int not null
   , w text not null
   , stop boolean default false
   , wlen int generated always as ( length( w ) ) stored
   , primary key( pk )
   , unique( n )
);

TRUNCATE pwc160.words;

INSERT INTO pwc160.words( n, w )
VALUES
( 1, 'One' )
, ( 2, 'Two' )
, ( 3, 'Three' )
, ( 4, 'Four' )
, ( 5, 'Five' )
, ( 6, 'Six' )
, ( 7, 'Seven' )
, ( 8, 'Eight' )
, ( 9, 'Nine' );

UPDATE pwc160.words
SET stop = true
WHERE n = 4;


CREATE OR REPLACE FUNCTION
pwc160.four_is_magic_plpgsql( needle int )
RETURNS SETOF text
AS $CODE$
DECLARE
    current_row pwc160.words%rowtype;
BEGIN

        SELECT *
        INTO current_row
        FROM pwc160.words
        WHERE n = needle;

        WHILE NOT current_row.stop  LOOP
              RETURN NEXT current_row.w || ' is ' || current_row.wlen;

              SELECT *
              INTO current_row
              FROM pwc160.words
              WHERE n = current_row.wlen;
        END LOOP;

        RETURN NEXT current_row.w || ' is magic';
        RETURN;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The Pl/PgSQL function selects the first item out of the table, then does the loop until a *stop* is found. The stop flag is stored into the table, just for ease of configuration.

<a name="task2plpgsql"></a>
## PWC 160 - Task 2 in PostgreSQL Pl/PgSQL

A little more complex implementation, since there is the need to `unnest` every array slice in order to present the parts as tables and be able to `sum` them:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc160.equilibrium_plpgsql( A int[] )
RETURNS int
AS $CODE$
DECLARE
   sum_a int;
   sum_b int;
BEGIN
    FOR idx IN 1 .. array_length( A, 1 )  LOOP
        SELECT sum( n )
        INTO sum_a
        FROM unnest( A[ 0 : idx - 1 ] ) n;

        SELECT sum( n )
        INTO sum_b
        FROM unnest( A[ idx : array_length( A, 1 ) ] ) n;

        IF sum_a = sum_b THEN
           RETURN idx;
        END IF;
    END LOOP;

    RETURN -1;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


Anyway, the logic is the same as in the other implementations.
