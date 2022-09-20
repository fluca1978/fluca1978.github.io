---
layout: post
title:  "Perl Weekly Challenge 179: graphs and spelled numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 179: graphs and spelled numbers

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 179](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0179/){:target="_blank"}.

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
## PWC 179 - Task 1

Implement a *number spelling* application so that, given an integer value, it can print out its english name in a ranking list.



<br/>
<br/>
```raku
sub MAIN( Int $n where { 0 < $n < 100 } ) {

    my @units = 'first',
                'second',
                'third',
                'foruth',
                'fifth',
                'sixth',
                'seventh',
                'eigth',
                'nineth',
                'tenth';

    my @teens = 'eleven', 'twelve', 'thriteen', 'fourteen', 'fifteen', 'sixteen', 'seventeen',
                'eigtheen', 'nineteen';
    my @non-teens = 'twenty', 'thirty', 'fourty', 'fifty', 'sixty', 'seventy', 'eighty', 'ninety';

    given ( $n ) {
        when $_ <= 10 { @units[ $n - 1 ].say; }
        when $_ < 20  { @teens[ ( $n - 1 ) % 10 ].say; }
        when $_ >= 20 {
            say @non-teens[ ( $n / 10 ).Int - 2 ]
                ~ ( $n %% 10 ?? '' !! @units[ ( $n % 10 ) - 1 ] );
        }
        default { "Cannot spell $n".say; }
    }

}

 ```
<br/>
<br/>

For simplicity I assume the number is less than `100`. I create a few arrays to keep track of words for values within `1` and `10`, between `11` and `19`, and all multiplies of `10` up to `90`.
<br/>
Then I check where the input value `$n` is within, and get out the exact word or compose one out of multiple words.


<a name="task2"></a>
## PWC 179 - Task 2

It took me a while to understand what I had to do, but to keep it simple, it is a way to produce a graph out of of a number array.

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems } ) {
    my @symbols = '▁' ... '█';
    my ($min, $max) = @n.min, @n.max;
    my @graph = @n.map: { ( $_ - $min ) / ( $max - $min ) * @n.elems };
    @symbols[ @graph ].join.say;
}

```
<br/>
<br/>

The `@symbols` array (or better, sequence) contains all the UTF-8 chars to presents a bar based graph.
The `@graph` array contains the values the user provided mapped into a position into the array of symbols. Therefore, I just need to print out the slice of the `@symbols` to produce the graph.

<a name="task1plperl"></a>
## PWC 179 - Task 1 in PostgreSQL PL/Perl

Very similar solution to the Raku one:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc179.task1_plperl( int )
RETURNS text
AS $CODE$
    my @units = qw/
                first
                second
                third
                foruth
                fifth
                sixth
                seventh
                eigth
                nineth
                tenth
                /;

    my @teens = qw /
              eleven
              twelve
              thriteen
              fourteen
              fifteen
              sixteen
              seventeen
              eigtheen
              nineteen
              /;
    my @non_teens = qw/
                    twenty
                    thirty
                    fourty
                    fifty
                    sixty
                    seventy
                    eighty
                    ninety
                    /;

     my ( $n ) = @_;
     return 'Cannot spell'                 if ( $n >= 100 );
     return $units[ $n - 1 ]               if ( $n <= 10 );
     return $teens[ ( $n - 1 ) % 10  ]     if ( $n > 10 && $n < 20 );
     return $non_teens[ ( $n / 10 ) - 2 ]  if ( $n >= 20 && $n % 10 == 0 );
     return $non_teens[ ( $n / 10 ) - 2 ] . $units[ ( $n % 10 ) - 1 ] if ( $n > 20 );


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 179 - Task 2 in PostgreSQL PL/Perl


A reimplementation of the Raku approach:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc179.task2_plperl( int[] )
RETURNS text
AS $CODE$

   my ($n) = shift;
   my @n;

   my @symbols = map {chr($_)} (0x2581..0x2588);

   my ($min, $max) = (-1,-1);

   # compute min and max over the values
   for my $current ( @$n ) {
       $max = $current if ( $current > $max );
       $min = $current if ( $min == -1 || $current < $min );
       push @n, $current;
   }

   my @graph = map { ( $_ - $min ) / ( $max - $min ) * scalar( @$n ) } @$n;
   return join( '', @symbols[ @graph ] );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 179 - Task 1 in PostgreSQL PL/PgSQL

I decided to use a table, indexed by the numeric value, to keep track of the basic dictionary of words. Then the function tries to extract a word from the table. If no words is found (i.e., `FOUND` is false), the word must be composed out of a computation.

<br/>
<br/>

``` sql
CREATE TABLE IF NOT EXISTS
pwc179.number2words
(
        v int PRIMARY KEY
        , t text
);

TRUNCATE pwc179.number2words;

INSERT INTO pwc179.number2words
VALUES
  ( 1, 'first' )
, ( 2, 'second' )
, ( 3, 'third' )
, ( 4, 'fourth' )
, ( 5, 'fifth' )
, ( 6, 'sixth' )
, ( 7, 'seventh' )
, ( 8, 'eigth' )
, ( 9, 'nineth' )
, ( 10, 'tenth' )
, ( 11, 'eleventh' )
, ( 12, 'twelveth' )
, ( 13, 'thirteenth' )
, ( 14, 'fourtineenth' )
, ( 15, 'fifteenth' )
, ( 16, 'sixteenth' )
, ( 17, 'seventeenth' )
, ( 18, 'eigthteenth' )
, ( 19, 'nineteenth' )
, ( 20, 'twentyth' )
, ( 30, 'thirtyth' )
, ( 40, 'fourtyth' )
, ( 50, 'fiftyth' )
, ( 60, 'sixtyth' )
, ( 70, 'seventyth' )
, ( 80, 'eightyth' )
, ( 90, 'ninetyth' );


CREATE OR REPLACE FUNCTION
pwc179.task1_plpgsql( n int )
RETURNS TEXT
AS $CODE$

DECLARE
        w text;
        s text;
BEGIN
        SELECT t
        INTO w
        FROM pwc179.number2words
        WHERE v = n;

        IF FOUND THEN
           RETURN w;
        ELSE
           -- not found, compose the word
           SELECT t
           INTO w
           FROM pwc179.number2words
           WHERE v = ( n / 10 )::int;

           SELECT t
           INTO s
           FROM pwc179.number2words
           WHERE v = ( n % 10 )::int;

           RETURN replace( w, 'th', 'ty') || s;
        END IF;
END

$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

In the case the word has to be composed, I `replace` the suffix with the correct one before concatenating.


<a name="task2plpgsql"></a>
## PWC 179 - Task 2 in PostgreSQL PL/PgSQL

Like the previous task, I created a table to contains the symbols, and then extract the values out of the symbol table.

<br/>
<br/>

``` sql
CREATE TABLE IF NOT EXISTS pwc179.symbols
(
        v int PRIMARY KEY
        , s text
);

TRUNCATE pwc179.symbols;

INSERT INTO pwc179.symbols
VALUES
( 0, '▁')
,(1, '▁')
,(2,'▂')
,(3, '▃')
,(4, '▄')
,(5, '▅')
,(6, '▆')
,(7,'▇')
,(8, '█')
,(9, '█');


CREATE OR REPLACE FUNCTION
pwc179.task2_plpgsql( n int[] )
RETURNS text
AS $CODE$
DECLARE
        c int;
        t text;
        tt text;
        scale_max int;
        scale_min int;
        scale_count int;
BEGIN
     t := '';

     SELECT min(v), max(v), count(v)
     INTO scale_min, scale_max, scale_count
     FROM pwc179.symbols;

     FOREACH c IN ARRAY n LOOP
         SELECT s
         INTO   tt
         FROM pwc179.symbols
         WHERE v = ( ( c - scale_min ) / ( scale_max - scale_min ) );
         t := t || tt;
     END LOOP;

     RETURN t;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

One interesting thing to note here is that, like Perl and Raku, PL/PgSQL allows for multiple variable assignation via the `SELECT INTO` statement, so it is quite easy to compute the min, max and counting values out of the symbol tables.
<br/>
Then I extract the symbol and concatenate to the `t` string, that is last returned.
