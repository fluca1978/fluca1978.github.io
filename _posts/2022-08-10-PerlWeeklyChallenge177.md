---
layout: post
title:  "Perl Weekly Challenge 177: damn numbers!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 177: damn numbers!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 177](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0177/){:target="_blank"}.

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
## PWC 177 - Task 1

Implement a *Damn aghoritm checker*, an application that can check if a number is correct using the Damn alghoritm, that is built on top of a permutation table.



<br/>
<br/>
```raku
sub MAIN( Int $number-to-check, Bool :$verbose = True ) {

    my @check-table =
                     [ 0, 3, 1, 7, 5, 9, 8, 6, 4, 2 ],
                     [ 7, 0, 9, 2, 1, 5, 4, 8, 6, 3 ],
                     [ 4, 2, 0, 6, 8, 7, 1, 3, 5, 9 ],
                     [ 1, 7, 5, 0, 9, 8, 3, 4, 2, 6 ],
                     [ 6, 1, 2, 3, 0, 4, 5, 9, 7, 8 ],
                     [ 3, 6, 7, 4, 2, 0, 9, 5, 8, 1 ],
                     [ 5, 8, 6, 9, 7, 2, 0, 1, 3, 4 ],
                     [ 8, 9, 4, 5, 3, 6, 2, 0, 1, 7 ],
                     [ 9, 4, 3, 8, 6, 1, 7, 2, 0, 5 ],
                     [ 2, 5, 8, 1, 4, 3, 6, 7, 9, 0 ];

    my $interim = 0;

    my @digits = $number-to-check.comb;
    my $check-digit = @digits[ * - 1 ];

    "Number $number-to-check will be checked as { @digits.join } against check digit $check-digit".say if $verbose;

    for @digits -> $column {
        "Digit $column (column) with interim (row) $interim => { @check-table[ $interim ][$column]}".say if $verbose;
        $interim = @check-table[ $interim ][$column];
    }

    "Number $number-to-check with last interim is $interim".say if $verbose;
    '1'.say and exit if $interim == 0;
    '0'.say;

}

 ```
<br/>
<br/>

Once you have the table, the alghoritm is quite simple: iterate over the table entries depending on every digit you get at every step. If the final result is zero, the number is correct.


<a name="task2"></a>
## PWC 177 - Task 2

Find *cyclop palindromes*, those palindrome numbers that have a zero in the middle.

<br/>
<br/>
```raku
sub MAIN( int $limit = 20 ) {

    my @cyclops = lazy gather {
        for 100 .. Inf {
            # skip numbers that have not an odd count of digits
            next if $_.Str.chars %% 2;

            # skip if the number does not have a middle zero
            next if $_.comb[ ( $_.Str.chars / 2 ).Int ] != 0;

            # skip if it is not palindrome
            next if $_.Str != $_.Str.flip;

            take $_;
        }
    };

    @cyclops[ 0 .. $limit ].join( "\n" ).say;

}

```
<br/>
<br/>

The idea is simple: for evrey number I produce I do skip non palindrome ones, those that do not have a zero in the middle of their digits, and those that have an even number of digits.

<a name="task1plperl"></a>
## PWC 177 - Task 1 in PostgreSQL PL/Perl

Very similar solution to the Raku one:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc177.task1_plperl( int )
RETURNS BOOLEAN
AS $CODE$

   my ( $number_to_check ) = @_;
    my $check_table = [
                     [ 0, 3, 1, 7, 5, 9, 8, 6, 4, 2 ],
                     [ 7, 0, 9, 2, 1, 5, 4, 8, 6, 3 ],
                     [ 4, 2, 0, 6, 8, 7, 1, 3, 5, 9 ],
                     [ 1, 7, 5, 0, 9, 8, 3, 4, 2, 6 ],
                     [ 6, 1, 2, 3, 0, 4, 5, 9, 7, 8 ],
                     [ 3, 6, 7, 4, 2, 0, 9, 5, 8, 1 ],
                     [ 5, 8, 6, 9, 7, 2, 0, 1, 3, 4 ],
                     [ 8, 9, 4, 5, 3, 6, 2, 0, 1, 7 ],
                     [ 9, 4, 3, 8, 6, 1, 7, 2, 0, 5 ],
                     [ 2, 5, 8, 1, 4, 3, 6, 7, 9, 0 ]
                     ];

    my $interim = 0;

    for my $column ( split( //, $number_to_check ) ) {
        $interim = $check_table->[ $interim ][ $column ];
    }

    return 1 if $interim == 0;
    return 0;


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 177 - Task 2 in PostgreSQL PL/Perl


A reimplementation of the Raku approach:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc177.task2_plperl( int )
RETURNS SETOF INT
AS $CODE$

my ( $limit ) = @_;

for my $current ( 100 .. 999999 ) {
    next if length( $current ) % 2 == 0;
    next if $current != reverse( $current );
    next if ( split( //, $current) )[ length( $current ) / 2 ] != 0;


    $limit-- and return_next( $current );
    last if $limit <= 0;
}

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 177 - Task 1 in PostgreSQL PL/PgSQL

Here I use a temporary table to store the *interim* values for the check table. Yeah, filling in the table is boring, but once it is done, the function can reduce its core at a few lines.
<br/>
I decided to use a `TEMPORARY TABLE` to store the values, indexing them by *rown* and *column* so that the subsequent selection will result easier. The table has the clause `ON COMMIT DROP` that means it will be deleted once the function (that runs in a transaction) ends.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc177.task1_plpgsql( n int )
RETURNS BOOLEAN
AS $CODE$
DECLARE
        interim int := 0;
        col     int := 0;
BEGIN

        CREATE TEMPORARY TABLE
        t_check( r int, c int, v int )
        ON COMMIT DROP;

        INSERT INTO t_check( r, c, v )
      VALUES
      ( 0, 0, 0),
      ( 0, 1,  3),
      ( 0, 2,  1),
      ( 0, 3, 7),
      ( 0, 4, 5),
      ( 0, 5, 9),
      ( 0, 6, 8),
      ( 0, 7, 6),
      ( 0, 8, 4),
      ( 0, 9, 2),
      ( 1,0,  7),
      ( 1,1, 0),
      ( 1,2, 9),
      ( 1,3, 2),
      ( 1,4, 1),
      ( 1,5,  5),
      ( 1,6, 4),
      ( 1,7,  8),
      ( 1,8,  6),
      ( 1,9, 3),
      ( 2,0, 4),
      ( 2,1, 2),
      ( 2,2, 0),
      ( 2,3, 6),
      ( 2,4, 8),
      ( 2,5, 7),
      ( 2,6, 1),
      ( 2,7, 3),
      ( 2,8,  5),
      ( 2,9,  9),
      ( 3,0,  1),
      ( 3,1,  7),
      ( 3,2,   5),
      ( 3,3,  0),
      ( 3,4,  9),
      ( 3,5,  8),
      ( 3,6,  3),
      ( 3,7,  4),
      ( 3,8,   2),
      ( 3,9,  6),
      ( 4,0,  6),
      ( 4,1,  1),
      ( 4,2,  2),
      ( 4,3,  3),
      ( 4,4,  0),
      ( 4,5,  4),
      ( 4,6,  5),
      ( 4,7,  9),
      ( 4,8,  7),
      ( 4,9,  8),
      ( 5,0, 3),
      ( 5,1, 6),
      ( 5,2, 7),
      ( 5,3, 4),
      ( 5,4, 2),
      ( 5,5, 0),
      ( 5,6, 9),
      ( 5,7, 5),
      ( 5,8, 8),
      ( 5,9, 1) ,
      ( 6,0, 5),
      ( 6,1, 8),
      ( 6,2, 6),
      ( 6,3, 9),
      ( 6,4, 7),
      ( 6,5, 2),
      ( 6,6, 0),
      ( 6,7, 1),
      ( 6,8, 3),
      ( 6,9, 4) ,
      ( 7,0, 8),
      ( 7,1, 9),
      ( 7,2, 4),
      ( 7,3, 5),
      ( 7,4, 3),
      ( 7,5, 6),
      ( 7,6, 2),
      ( 7,7, 0),
      ( 7,8, 1),
      ( 7,9, 7) ,
      ( 8,0, 9),
      ( 8,1, 4),
      ( 8,2, 3),
      ( 8,3, 8),
      ( 8,4, 6),
      ( 8,5, 1),
      ( 8,6, 7),
      ( 8,7, 2),
      ( 8,8, 0),
      ( 8,9, 5) ,
      ( 9,0,  2),
      ( 9,1,  5),
      ( 9,2,  8),
      ( 9,3,  1),
      ( 9,4,  4),
      ( 9,5,  3),
      ( 9,6,  6),
      ( 9,7,  7),
      ( 9,8,  9),
      ( 9,9,  0)
         ;


         FOR col IN SELECT regexp_split_to_table( n::text, '' ) LOOP
                 SELECT v
                 INTO interim
                 FROM t_check
                 WHERE c = col
                 AND   r = interim;
         END LOOP;

         IF interim = 0 THEN
            RETURN TRUE;
         ELSE
           RETURN FALSE;
         END IF;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The usage of `regexp_split_to_table` allows me to iterate quickly over the digits of the incoming number.


<a name="task2plpgsql"></a>
## PWC 177 - Task 2 in PostgreSQL PL/PgSQL

Similar implementation to the Raku solution, except that I use `reverse` to check if the number is palindrome and `substring` to get its midway digit (*pay atetntion: in SQL strings starts at character `1`*):

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc177.task2_plpgsql( l int default 20 )
RETURNS SETOF INT
AS $CODE$
DECLARE
        i int;
BEGIN
        FOR i IN 100 .. 99999 LOOP
            IF i::text <> reverse( i::text ) THEN
               -- not palindrome
               CONTINUE;
            END IF;

            IF length( i::text ) % 2 = 0 THEN
               -- even length
               CONTINUE;
            END IF;

            IF substring( i::text, length( i::text ) / 2 + 1, 1 ) <> '0' THEN
               CONTINUE;
            END IF;

            RETURN NEXT i;
            l := l - 1;
            EXIT WHEN l = 0;

        END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
