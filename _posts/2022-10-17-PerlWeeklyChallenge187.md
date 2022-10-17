---
layout: post
title:  "Perl Weekly Challenge 187: overlapping days and numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 187: overlapping days and numbers

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 187](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0187/){:target="_blank"}.

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
## PWC 187 - Task 1

Given days in the format `DD-MM`, assuming they belong to the very same year, compute the number of days overlapping.

<br/>
<br/>
```raku
sub MAIN( Str $start-foo, Str $end-foo, Str $start-bar, Str $end-bar ) {

    $start-foo ~~ / ^ ( \d ** 2 ) '-' ( \d ** 2 ) $ /;
    my $foo-days-start = DateTime.new: year => 2022, day => $/[0], month => $/[1];
    $end-foo ~~ / ^ (\d ** 2) '-' (\d **2) $ /;
    my $foo-days-end = DateTime.new: year => 2022, day => $/[0], month => $/[1];

    $start-bar ~~ / ^ ( \d ** 2 ) '-' ( \d ** 2 ) $ /;
    my $bar-days-start = DateTime.new: year => 2022, day => $/[0], month => $/[1];
    $end-bar ~~ / ^ ( \d ** 2 ) '-' ( \d ** 2 ) $ /;
    my $bar-days-end = DateTime.new: year => 2022, day => $/[0], month => $/[1];

    my $days = 0;
    my $start = $foo-days-start >= $bar-days-start ?? $bar-days-start !! $foo-days-start;
    my $end   = $foo-days-end >= $bar-days-end ?? $foo-days-end !! $bar-days-end;

    while ( $start < $end ) {
    	  $days++ if ( $start >= $foo-days-start
	  && $start <= $foo-days-end
	  && $start >= $bar-days-start
	  && $start <= $bar-days-end );
	  $start .= later( days => 1 );
    }

    say $days;
}

 ```
<br/>
<br/>

There is much more code to pars the dates than to do the effective application. The idea is to go one day after the other and see if such day is within both the sets.


<a name="task2"></a>
## PWC 187 - Task 2
Given a list of numbers, find out the triplets that satisfy a particular condition, having the sum as its maximum value.
It is not clear to me if the numbers have to be sort or not, so I assume that I need to find out the triplet proceeding in the exact sequence of numbers given as input.

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems } ) {

    my $max = 0;
    my @triplet;
    for 0 ..^ @n.elems -> $index-a {
    	my $a = @n[ $index-a ].Int;

	for $index-a ^..^ @n.elems -> $index-b {
	    my $b = @n[ $index-b ].Int;

	    for $index-b ^..^ @n.elems -> $index-c {
	    	my $c = @n[ $index-c ].Int;

		if ( $max <= $a + $b + $c
		     && ( $a + $b ) > $c
		     && ( $b + $c ) > $a
		     && ( $a + $c ) > $b ) {
		     @triplet = $a, $b, $c;
		     $max = $a + $b + $c;
		}
	    }
	}
    }

    @triplet.sort.say if ( @triplet );
}


```
<br/>
<br/>


Essentially I do three nested loops and see if the conditions are matched.

<a name="task1plperl"></a>
## PWC 187 - Task 1 in PostgreSQL PL/Perl

Same approach as Raku, but using `DateTime` here requires a little more date-math since operators are not overloaded.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc187.task1_plperl( text, text, text, text )
RETURNS INT
AS $CODE$
use DateTime;
my ( $foo_start, $foo_end, $bar_start, $bar_end ) = @_;
my @dates;

for ( $foo_start, $foo_end, $bar_start, $bar_end ) {
  $_ =~ / ^ (\d{2}) - (\d{2}) $ /x;
  push @dates, DateTime->new( year => 2022, day => $1, month => $2 );
}

my $days = 0;

if ( DateTime->compare( $dates[ 0 ], $dates[ 3 ] ) > 0
     || DateTime->compare( $dates[ 1 ], $dates[ 2 ] ) < 0 ) {
  # one ends before the begin of the other or viceversa
  $days = 0;
}
else {
  my ( $start, $end );
  $start = DateTime->compare( $dates[ 0 ], $dates[ 2 ] ) <= 0 ? $dates[ 0 ] : $dates[ 2 ];
  $end   = DateTime->compare( $dates[ 1 ], $dates[ 3 ] ) <= 0 ? $dates[ 1 ] : $dates[ 3 ];

  while ( DateTime->compare( $start, $end ) <= 0 ) {
    $days++ if ( DateTime->compare( $start, $dates[ 0 ] ) >= 0
		 && DateTime->compare( $start, $dates[ 1 ] ) <= 0
		 && DateTime->compare( $start, $dates[ 2 ] ) >= 0
		 && DateTime->compare( $start, $dates[ 3 ] ) <= 0 );
    $start = $start->add( days => 1 );
  }
}



return $days;

$CODE$
LANGUAGE plperlu;


```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 187 - Task 2 in PostgreSQL PL/Perl

Same implementation as in Raku.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc187.task2_plperl( int[] )
RETURNS int[]
AS $CODE$
my ( @n ) = $_[0]->@*;
my @triplets;
my $max = 0;


for my $index_a ( 0 .. $#n ) {
  my $a = $n[ $index_a ];

  for my $index_b ( ( $index_a + 1 ) .. $#n ) {
    my $b = $n[ $index_b ];

    for my $index_c ( ( $index_b + 1 ) .. $#n ) {
      my $c = $n[ $index_c ];

      if ( $max <= $a + $b + $c
	  && ( $a + $b ) > $c
	  && ( $b + $c ) > $a
	  && ( $a + $c ) > $b ) {

	@triplets = ( $a, $b, $c );
	$max = $a + $b + $c;
      }
    }
  }
}


return [@triplets];

$CODE$
LANGUAGE plperl;


```
<br/>
<br/>



<a name="task1plpgsql"></a>
## PWC 187 - Task 1 in PostgreSQL PL/PgSQL

Here I use `date` as the input, so I can quickly do the math within the function.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc187.task1_plpgsql( foo_start date, foo_end date, bar_start date, bar_end date )
RETURNS int
AS $CODE$
DECLARE
	day_start date;
	day_end date;
	day_count int := 0;
BEGIN

	IF NOT (foo_start, foo_end) OVERLAPS (bar_start, bar_end) THEN
	   RETURN 0;
	END IF;

	IF foo_start > bar_start THEN
	   day_start := bar_start;
	ELSE
	   day_start := foo_start;
	END IF;

	IF foo_end > bar_start THEN
	   day_end := foo_end;
	ELSE
	  day_end := bar_end;
        END IF;

	WHILE day_start <= day_end LOOP
	      IF day_start >= foo_start AND day_start <= foo_end AND day_start >= bar_start AND day_end <= bar_end THEN
	      	 day_count := day_count + 1 ;
              END IF;
	      day_start := day_start + 1;
	END LOOP;

	RETURN day_count;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>




<a name="task2plpgsql"></a>
## PWC 187 - Task 2 in PostgreSQL PL/PgSQL

Too lazy, let's call PL/Perl to do all the math.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc187.task2_plpgsql( a int[] )
RETURNS int[]
AS $CODE$
BEGIN
	RETURN pwc187.task2_plperl( a );
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>
