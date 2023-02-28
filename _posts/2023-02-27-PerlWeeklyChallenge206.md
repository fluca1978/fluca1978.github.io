---
layout: post
title:  "Perl Weekly Challenge 206: hard times!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 206: hard times!

This post presents my solutions to the [Perl Weekly Challenge 206](https://perlweeklychallenge.org/blog/perl-weekly-challenge-206/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 206 - Task 1 - Raku](#task1)
- [PWC 206 - Task 2 - Raku](#task2)
- [PWC 206 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 206 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 206 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 206 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 206 - Task 1 - Raku Implementation

The first task was about computing the lowest difference between an array of times, expressed as `HH::MI` strings.
At first, I thought at `DateTime` but unluckily I was on a machine were the Raku documentation was absent and the online manual `https://docs.raku.org` was suffering too.
So, I had to implement it on my own.

<br/>
<br/>
```raku
sub diff ( $start, $end ) {
    my ( $start-hours, $start-mins ) = $start.chomp.split( ':' );
    my ( $end-hours, $end-mins ) = $end.chomp.split( ':' );

    if ( $start-hours == 0 ) {
	$start-hours = 23;
	$start-mins += 60;
    }

    if ( $end-hours == 0 ) {
	$end-hours = 23;
	$end-mins += 60;
    }

    my $diff-hours = abs( $end-hours - $start-hours );
    my $diff-mins  = abs( $end-mins - $start-mins ) % 60;

    return $diff-hours * 60 + $diff-mins;

}

sub MAIN( :$verbose = True, *@times where { @times.grep( * ~~ / ^ \d ** 2 ':' \d ** 2 $ /  ).elems == @times.elems } ) {

    my %diffs;
    %diffs{ diff( $_[ 1 ], $_[ 0 ] ) } = [ $_[0], $_[1] ] for @times.sort.combinations( 2 );

    %diffs.keys.map( *.Int ).min.say;
    %diffs{ %diffs.keys.map( *.Int ).min }.join( ' - ' ).say if ( $verbose );
}

```
<br/>
<br/>

The function `diff` accepts two time strings, computes the numeric values for hours and minutes and then returns the difference expressed as the number of minutes. There is a little trick in there: if the hour is expressed as `00:MI` I *downgrade* it to `23` and add sixty minutes. This allows me to compare every time without having to worry about the round clock.
<br/>
In the `MAIN` I create a `%diffs` hash keyed by the difference in minutes between every pair (`combinations(2)`) after the list has been `sorted`, so that I can consider *one way difference*. Last, I simply print the minimium key (as an integer, because it is managed as a string).


<a name="task2"></a>
## PWC 206 - Task 2 - Raku Implementation

Given an array of integers, consider all the pairs and sum the minimum of every pair, then get the overall minimum sum.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.elems %% 2 && @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my @sums;

    for @list.permutations {
    	for $_.rotor( 2 ) -> $a, $b {
	      @sums.push: sum( min( $a ) + min( $b ) );
	    }
    }

    @sums.min.say;
}

```
<br/>
<br/>

The above is a very unefficient solution, since it exploits all the `permutations` of the input array and consumes its as slices of two elements. When I was developing the PL/Perl solution a trick came into my mind.

# PL/Perl Implementations

A similar implementation to the Raku one: I use an anonymous subroutine to compute the difference between time strings.

<a name="task1plperl"></a>
## PWC 206 - Task 1 - PL/Perl Implementation

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc206.task1_plperl( text[] )
RETURNS int
AS $CODE$
   my ( $times ) = @_;
   my $computations = {};
   my $min;

   my $diff = sub {
      my ( $start, $end ) = @_;
      $start =~ /^(\d{2}):(\d{2})$/;
      my ( $start_hours, $start_mins ) = ( $1, $2 );

      $end =~ /^(\d{2}):(\d{2})$/;
      my ( $end_hours, $end_mins ) = ( $1, $2 );

      if ( $start_hours == 0) {
      	 $start_hours = 23;
	 $start_mins += 60;
      }

      if ( $end_hours == 0) {
      	 $end_hours = 23;
	 $end_mins += 60;
      }

      return abs( $end_hours - $start_hours ) * 60 + abs( $end_mins - $start_mins ) % 60;
   };


   for my $begin ( sort $times->@* ) {
       for my $end ( sort $times->@* ) {
       	   next if ( $begin eq $end );

	   my $difference = $diff->( $end, $begin );
	   $computations->{ $difference } = [ $begin, $end ];
	   $min = $difference if ( ! $min || $difference < $min );
       }
   }

   elog(INFO, "Min is $min minutes from " . join( ',', $computations->{ $min }->@* ) );
   return $min;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 206 - Task 2 - PL/Perl Implementation

Here I had an idea: if I sort the list of integers and then take one element out of two, I can sum all the mimum values of every pair and get, in a straight manner, the overall minimu sum.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc206.task2_plperl( int[] )
RETURNS int
AS $CODE$
  my ( $list ) = sort $_[ 0 ];
  my $sum = 0;

  while ( $list->@* ) {
  	$sum += shift $list->@*;
	shift $list->@*;
  }

  return $sum;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 206 - Task 1 - PL/PgSQL Implementation

Luckily, a difference between two `time` values is another `time` value, therefore it handles the difference right.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc206.task1_plpgsql( t text[] )
RETURNS time
AS $CODE$

DECLARE
	m time;
	t1 text;
	t2 text;
BEGIN

	FOREACH t1 IN ARRAY t LOOP
		FOREACH t2 IN ARRAY t LOOP
			IF t1 = t2 THEN
			   CONTINUE;
			END IF;

			IF m IS NULL OR ( t2::time - t1::time ) < m THEN
			   m := ( t2::time - t1::time );
			END IF;

		END LOOP;
	END LOOP;

	RETURN m;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 206 - Task 2 - PL/PgSQL Implementation

A single query to do the same as the PL/Perl implementation. I build a temporary table, using a CTE, where every row has a number associated to it by means of `row_number()`. Then I select all the odd rows, that represent the minimum value of the sorted input array, and the sum is the overall minimum.

<br/>
<br/>
```sql

CREATE OR REPLACE FUNCTION
pwc206.task2_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
   res int;
BEGIN
	WITH data AS (
    	    SELECT v, row_number() OVER ( ORDER BY v ) r
	    FROM unnest( l ) v
	)
	SELECT sum( v )
	INTO res
	FROM data
	WHERE r % 2 <> 0
	;

	RETURN res;
END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>
