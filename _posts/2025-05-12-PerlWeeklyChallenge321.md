---
layout: post
title:  "Perl Weekly Challenge 321: Task 2 simpler than Task 1"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 321: Task 2 simpler than Task 1

This post presents my solutions to the [Perl Weekly Challenge 321](https://perlweeklychallenge.org/blog/perl-weekly-challenge-321/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 321 - Task 1 - Raku](#task1)
- [PWC 321 - Task 2 - Raku](#task2)
- [PWC 321 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 321 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 321 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 321 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 321 - Task 1 in PL/Java](#task1pljava)
- [PWC 321 - Task 2 in PL/Java](#task2pljava)
- [PWC 321 - Task 1 in Python](#task1python)
- [PWC 321 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 321 - Task 1 - Raku Implementation

In the first task, given an array of integers, the script has to provide the min intermediate average, as computed from progressively average the min and max value left in the array.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.elems %% 2 && @numbers.elems == @numbers.grep( * ~~ Int ).elems } ) {

    my @averages;
    for 0 ..^ @numbers.elems / 2 {
		@averages.push: ( [+] @numbers.sort[ $_, * - 1 - $_ ] ) / 2;
    }

    @averages.min.say;
}

```
<br/>
<br/>

The idea is quite simple: I iterate over the boundaries of the array, compute the average and store it into the `@averages` array. Then, print the min value.



<a name="task2"></a>
## PWC 321 - Task 2 - Raku Implementation

Given two strings, assume the `#` is a backspace and hence deletes the left character. Return `True` if the strings are the same.

<br/>
<br/>
```raku
sub MAIN( Str $left, Str $right ) {
    say $left.subst( / . <[#]> /, '', :g ) ~~ $right.subst( / . <[#]> /, '', :g );
}

```
<br/>
<br/>

A single line does suffice: if the strings, after having `sub`stituted the backspace chaacters, do match then return `True`.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 321 - Task 1 - PL/Perl Implementation

Similar implemenration to the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc321.task1_plperl( int[] )
RETURNS numeric
AS $CODE$

   my ( $numbers ) = @_;
   die "Must be even in size!" if ( $numbers->@* % 2 != 0 );

   $numbers = [ sort $numbers->@* ];

   my $min_average = undef;

   for ( 0 .. $numbers->@* - 1 ) {
       my $current = ( $numbers->@[ $_ ] + $numbers->@[ $numbers->@* - $_ - 1 ] ) / 2;
       $min_average = $current if ( ! $min_average || $min_average > $current );
   }

   return $min_average;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


This time, instead of having an array of averages, I keep directly the min value computed at each iteration.



<a name="task2plperl"></a>
## PWC 321 - Task 2 - PL/Perl Implementation

Similar implementation to the Raku one.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc321.task2_plperl( text, text )
RETURNS boolean
AS $CODE$

   my ( $left, $right ) = @_;

   $left  =~ s/ . [#]//xg;
   $right =~ s/ . [#]//xg;

   return $left eq $right ? 1 : 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 321 - Task 1 - PL/PgSQL Implementation

In this implementation I use a temporary table to store the computed averages, so that I can extract the min value from such table.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc321.task1_plpgsql( numbers int[] )
RETURNS numeric
AS $CODE$
DECLARE
	i int;
	result numeric;
	current int[];
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS averages( v numeric, mx int, mn int );
	TRUNCATE averages;

	FOR i IN 0 .. array_length( numbers, 1 ) / 2 - 1 LOOP

	    INSERT INTO averages( v, mx, mn )
	    SELECT ( min( x::int ) + max( x::int ) ) / 2::numeric
	    	   , max( x )::int
	    	   , min( x )::int

	    FROM unnest( numbers[ 1 + i : array_length( numbers, 1 ) - i ] ) x
	    ;
	END LOOP;

	SELECT min( v )
	INTO result
	FROM averages;

	RETURN result;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Note that the `unnest` works on array slice, not two single elements, but since I then apply `min` and `max`, I get the same result as having computed only two values.


<a name="task2plpgsql"></a>
## PWC 321 - Task 2 - PL/PgSQL Implementation

Use `regexp_replace` to do the trick, so that a single query does suffice.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc321.task2_plpgsql( l text, r text )
RETURNS boolean
AS $CODE$
   SELECT regexp_replace( l, '.[#]', '', 'g' ) = regexp_replace( r, '.[#]', '', 'g' );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 321 - Task 1 - PostgreSQL PL/Java Implementation

Iterative approach in Java, using a collection and keeping the min value computed at every iteration.


<br/>
<br/>
```java
    @Function( schema = "pwc321",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final Float task1_pljava( int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc321.task1_pljava" );

		if ( numbers.length % 2 != 0 )
		    throw new SQLException( "Must be even in size" );

		Float min_average = null;

		List<Integer> sorted = IntStream.of( numbers ).boxed().collect( Collectors.toList() );
		Collections.sort( sorted );

		for ( int i = 0; i < sorted.size(); i++ ) {
		    float current = ( sorted.get( i ) + sorted.get( sorted.size() - i - 1 ) ) / (float) 2;
		    if ( min_average == null || current < min_average )
				min_average = current;
		}

		return min_average;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 321 - Task 2 - PostgreSQL PL/Java Implementation

Using regular expression to the rescue.

<br/>
<br/>
```java
	@Function( schema = "pwc321",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final boolean task2_pljava( String left, String right ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc321.task2_pljava" );

	    return left.replaceAll( ".[#]", "" ).equals( right.replaceAll( ".[#]", "" ) );
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 321 - Task 1 - Python Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    numbers = list( sorted( list( map( int, args ) ) ) )
    min_average = None

    for i in range( 0, len( numbers ) ):
        current = ( numbers[ i ] + numbers[ len( numbers ) - i - 1 ] ) / 2
        if min_average is None or current < min_average:
            min_average = current

    return min_average


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 321 - Task 2 - Python Implementation

Use regular expressions to compare the strings.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_2( args ):
    left  = args[ 0 ]
    right = args[ 1 ]

    left  = re.sub( '.[#]', '', left )
    right = re.sub( '.[#]', '', right )

    return left == right


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
