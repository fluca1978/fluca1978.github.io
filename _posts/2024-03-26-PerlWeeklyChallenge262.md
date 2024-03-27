---
layout: post
title:  "Perl Weekly Challenge 262: another short one!"
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

# Perl Weekly Challenge 262: another short one!

This post presents my solutions to the [Perl Weekly Challenge 262](https://perlweeklychallenge.org/blog/perl-weekly-challenge-262/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 262 - Task 1 - Raku](#task1)
- [PWC 262 - Task 2 - Raku](#task2)
- [PWC 262 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 262 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 262 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 262 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 262 - Task 1 in PL/Java](#task1pljava)
- [PWC 262 - Task 2 in PL/Java](#task2pljava)
- [PWC 262 - Task 1 in Python](#task1python)
- [PWC 262 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 262 - Task 1 - Raku Implementation

The first task was about computing the max counting among positive and negative integers.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    ( @nums.grep( * >= 0 ).elems, @nums.grep( * < 0 ).elems ).max.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 262 - Task 2 - Raku Implementation

The second task was to compute how many pairs of numbers are in a given array so that one is leftmost of the other, and the indexes of their positions multiplied together are a multiple of a given value `k`.


<br/>
<br/>
```raku
sub MAIN( Int $k where { $k != 0 },
	  *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {

    my @pairs;

    for 0 ..^ @nums.elems -> $i {
		for $i ^..^ @nums.elems -> $j {
		    next if ( $i * $j ) !%% $k;
		    next if @nums[ $i ] != @nums[ $j ];
		    @pairs.push: [ $i, $j ];
		}
    }

    @pairs.elems.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 262 - Task 1 - PL/Perl Implementation

A very simple implementation using again `grep`.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc262.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;

  my ( $positives, $negatives ) =
    ( scalar( grep { $_ >= 0 } $nums->@* ),
      scalar( grep { $_ < 0 } $nums->@* ) );

  return $positives >= $negatives
  	   ? $positives
	   :  $negatives;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 262 - Task 2 - PL/Perl Implementation

A nested loop to solve the trick.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc262.task2_plperl( int, int[] )
RETURNS int
AS $CODE$

   my ( $k, $nums ) = @_;

   my @pairs;
   for my $i ( 0 .. $nums->@* - 1 ) {
       for my $j ( $i + 1 .. $nums->@* - 1 ) {
       	   next if ( $i * $j ) % $k != 0;
		   next if ( $nums->@[ $i ] != $nums->@[ $j ] );
		   push @pairs, [ $i, $k ];
       }
   }


   return scalar @pairs;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 262 - Task 1 - PL/PgSQL Implementation

A single query with a few subqueries to compute the max value.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc262.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$

   WITH positives AS (
   	SELECT count( v ) as c
	FROM unnest( nums ) v
	WHERE  v >= 0
	), negatives AS (
	SELECT count( v ) as c
	FROM unnest( nums ) v
	WHERE  v < 0
	)
	, b AS (
	  SELECT c as v FROM positives
	  UNION
	  SELECT c as v FROM negatives
	  )
   SELECT max( b.v )
   FROM b;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 262 - Task 2 - PL/PgSQL Implementation

A nested loop to count the pairs. Please note that in SQL the indexes starts from `1`, so the same input does not provide the same result as in other languages.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc262.task2_plpgsql( k int, nums int[] )
RETURNS int
AS $CODE$
DECLARE
	pairs_count int := 0;
BEGIN
	FOR i IN 1 .. array_length( nums, 1 )  LOOP
	    FOR j IN ( i + 1 ) .. array_length( nums, 1 )  LOOP
	    	IF nums[ i ] <> nums[ j ] THEN
		   CONTINUE;
		END IF;

		IF mod( i * j, k ) <> 0 THEN
		   CONTINUE;
		END IF;

		pairs_count := pairs_count + 1;
	    END LOOP;
	END LOOP;

	RETURN pairs_count;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 262 - Task 1 - PostgreSQL PL/Java Implementation

Very simple implementation with a single loop that counts the values depending on their "type".

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc262",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc262.task1_pljava" );

		int positives = 0;
		int negatives = 0;

		for ( int current : nums )
		    if ( current >= 0 )
				positives++;
		    else
				negatives++;

		return positives >= negatives ? positives : negatives;

    }
}

```
<br/>
<br/>


This can be solved also using *streams*, for example:

<br/>
<br/>
```java
    public static final int task1_pljava( int[] nums ) throws SQLException {
		final int[] results = new int[]{ 0, 0 };

		Arrays.stream( nums )
		    .forEach( current -> {
			    int index = current >= 0 ? 0 : 1;
			    results[ index ]++;
			} );

		return results[ 0 ] >= results[ 1 ] ? results[ 0 ] : results[ 1 ];

    }
```
<br/>
<br/>

The idea is:
- convert the `nums` array to a stream using `Arrays.stream`
- `forEach` value, placed into `current` I check if the value is positive or not, and extract an `index`, either zero or one
- increment the location of `results` accordingly, with `results[ 0 ]` that represents the positive values and `results[ 1 ]` that represents negative values.

<a name="task2pljava"></a>
## PWC 262 - Task 2 - PostgreSQL PL/Java Implementation

A nested loop even in this implementation.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc262",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( int k, int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc262.task2_pljava" );

		int pairs = 0;
		for ( int i = 0; i < nums.length; i++ )
		    for ( int j = i + 1; j < nums.length; j++ )
				if ( nums[ i ] != nums[ j ]
				     || ( i * j ) % k != 0 )
				    continue;
				else
				    pairs++;

		return pairs;

    }
}

```
<br/>
<br/>


It is possible to implement this with *stream*, for example:

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc262",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( final int k, final int[] nums ) throws SQLException {

		final int pairs[] = new int[]{ 0 };
		IntStream.range( 0, nums.length )
		    .forEach( i -> {
			    IntStream.range( i + 1, nums.length )
				.forEach( j -> {
					if ( nums[ i ] == nums[ j ]
					     && ( i * j ) % k == 0 )
					    pairs[ 0 ]++;
				    } );
			} );


		return pairs[ 0 ];

    }
}

```
<br/>
<br/>

The idea is as follows:
- the `pairs` array is a monodimensional array used only to access a counter from within the streams;
- the outer `IntStream.range` loops on the `i` variable, and `forEach` value opens another stream;
- the inner `IntStream.range` loops over the `i+1` value on `j` and `forEach` value tests if the array values are the same and satisfy the aim of the task
- if the current pair is fine, the counter is incremented.


# Python Implementations

<a name="task1python"></a>
## PWC 262 - Task 1 - Python Implementation

Same approach used in Java here: counting two types of values, but this time with a `filter` (like `grep` in Perl).

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    nums = list( map( int, args ) )
    positives = len( list( filter( lambda x: True if x >= 0 else False, nums ) ) )
    negatives = len( list( filter( lambda x: True if x < 0  else False, nums ) ) )

    return positives if positives >= negatives else negatives


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 262 - Task 2 - Python Implementation

A nested loop.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    k     = int( args[ 0 ] )
    nums  = list( map( int, args[ 1: ] ) )
    pairs = []

    for i in range( 0, len( nums ) ):
        for j in range( i + 1, len( nums ) ):
            if nums[ i ] != nums[ j ] or ( i * j ) % k != 0:
                continue
            else:
                pairs.append( [ i, j ] )

    return len( pairs )

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
