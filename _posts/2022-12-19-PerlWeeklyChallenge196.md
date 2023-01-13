---
layout: post
title:  "Perl Weekly Challenge 196: Merry Christmas!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 196: Merry Christmas!

This post presents my solutions to the [Perl Weekly Challenge 196](https://perlweeklychallenge.org/blog/perl-weekly-challenge-196/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 196 - Task 1 - Raku](#task1)
- [PWC 196 - Task 2 - Raku](#task2)
- [PWC 196 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 196 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 196 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 196 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.


<a name="task1"></a>
## PWC 196 - Task 1 - Raku Implementation


In the first there was a given input list of integers, and I need to find out any triplet that respect the `1-2-3` rule, that is the leftmost must be less than middle one that, in turn has to be less than the rightmost number.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } )  {
    my @found;
    my $last = 0;

    for @list.rotor( 3, :partial ) -> $triplet {
	next if $triplet.elems != 3;
	@found.push: $triplet if ( $triplet[ 0 ] < $triplet[ 1 ] < $triplet[ 2 ] );
    }

    @found.join( "\n" ).say;
}

```
<br/>
<br/>

I use a `rotor` to extract triplets, and skip all of them that are not complete since the input array could not be a multiple of three.
Then I keep the triplet into the `@found` array only if the triplet satisfies the `1-2-3` rule. Note how easy and short it is to exrepss a double disequation in Raku!


<a name="task2"></a>
## PWC 196 - Task 2 - Raku Implementation

Again a list of integers, sorted this time. The task required to find out all the indexes that indicate a natural number sequence within the input list.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {

    my @ranges;
    my $start = -1;
    my $end   = -1;
    for 0 .. @list.elems {
		next if ! $_;
		next if $_ <= $end;

		$start = $_;
		$end   = $start;

		$end++ while ( $end < @list.elems &&  @list[ $end + 1 ] == @list[ $end ] + 1 );
		@ranges.push: [ $start, $end ] if ( $start < $end );
    }

    @ranges.join( "\n" ).say;
}

```
<br/>
<br/>


I iterate over the input `@list` and keep track of the current position into `$start`, then I try to increment the `$end`indg index checking if the righter value is increased by one with regard to the current one. If I do have two different indexes, I keep them as an array into the `@ranges` set of results, that I then print.


<a name="task1plperl"></a>
## PWC 196 - Task 1 - PL/Perl Implementation

Straightforward implementation like the Raku approach:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc196.task1_plperl( int[] )
RETURNS SETOF int[]
AS $CODE$

  my ( $array ) = $_[ 0 ];
  my $index = 1;

  while ( $index < $array->@* ) {
     my @triplet = ( $array->[ $index - 1 ], $array->[ $index ], $array->[ $index + 1 ] );
     $index += 2 and return_next( [ @triplet ] )  if ( $tripet[ 0 ] < $triplet[ 1 ]
                                && $triplet[ 1 ] < $triplet[ 2 ] );
     $index++;
  }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Note that I increment `$index` by two before returning a new result into the result set, so that when the routine resumes the `$index` is increased by another unit and skips therefore a triplet, if found.



<a name="task2plperl"></a>
## PWC 196 - Task 2 - PL/Perl Implementation

Similar implementation to the Raku one:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc196.task2_plperl( int[] )
RETURNS SETOF int[]
AS $CODE$
  my ( $array ) = $_[0];
  my ( $start, $end ) = ( 0, 0 );

  while ( $start < $array->@* ) {
     $end = $start;
     $end++ while ( $end < $array->@* &&   $array->[ $end + 1 ] == $array->[ $end ] + 1 );
     return_next( [ $start, $end ] ) if ( $end > $start );
     $start += $end + 1;
  }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task1plpgsql"></a>
## PWC 196 - Task 1 - PL/PgSQL Implementation

Same idea as the Raku and PL/Perl implementation: if the array triplets do the `1-2-3` rule I append them to the result set, otherwise I go forward in seeking for a new triplet.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc196.task1_plpgsql( l int[] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	last_index int := 0;
BEGIN

	FOR i IN 1 .. array_length(l,1) - 1 LOOP
	    IF i <= last_index THEN
	       CONTINUE;
	    END IF;

	    IF l[ i - 1 ] < l[ i ] AND l[ i ] < l[ i + 1 ] THEN
	       RETURN NEXT ARRAY[ l[i-1], l[i], l[i + 1] ]::int[];
	       last_index := i + 1;
	    END IF;
	END LOOP;
RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 196 - Task 2 - PL/PgSQL Implementation


A more verbose but conceptually identical implementation to the PL/Perl solution:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc196.task2_plpgsql( l int[])
RETURNS SETOF int[]
AS $CODE$
DECLARE
	c_start int := 0;
	c_end   int := 0;
BEGIN

	FOR i IN 0 .. array_length( l, 1 ) LOOP
	    IF i < c_end THEN
	       CONTINUE;
	    END IF;

	    c_start := i;
	    c_end := c_start;

	    WHILE c_end < array_length( l, 1 ) AND l[ c_end + 1 ] = l[ c_end ] + 1 LOOP
	    	  c_end := c_end + 1;
	    END LOOP;

	    IF c_start < c_end THEN
	       RETURN NEXT ARRAY[ c_start, c_end ]::int[];
	    END IF;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
