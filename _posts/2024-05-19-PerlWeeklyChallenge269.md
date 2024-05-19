---
layout: post
title:  "Perl Weekly Challenge 269: at the last time I did!"
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

# Perl Weekly Challenge 269: at the last time I did!

This post presents my solutions to the [Perl Weekly Challenge 269](https://perlweeklychallenge.org/blog/perl-weekly-challenge-269/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 269 - Task 1 - Raku](#task1)
- [PWC 269 - Task 2 - Raku](#task2)
- [PWC 269 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 269 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 269 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 269 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 269 - Task 1 in PL/Java](#task1pljava)
- [PWC 269 - Task 2 in PL/Java](#task2pljava)
- [PWC 269 - Task 1 in Python](#task1python)
- [PWC 269 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 269 - Task 1 - Raku Implementation

The first task was about finding out if, in a given array of integers, there are couples of values that once *logically OR-ed* made a value that in binary ends with at least one zero.
This means that the ending result must be **even**.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( { $_ ~~ Int && $_ > 0 } ).elems } ) {
    my @couples;
    for 0 ..^ @nums.elems -> $i {
	for $i ^..^ @nums.elems -> $j {
	    @couples.push: [ @nums[ $i ], @nums[ $j ] ] if ( ( @nums[ $i ] +| @nums[ $j ] ) %% 2 );
	}
    }

    @couples.join( "\n" ).say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 269 - Task 2 - Raku Implementation

The second task was about splitting a given array of numbers into two different ones, when the next element is inserted in the first array or in the second depepending on which arrays has the greatest last element. In the end, the arrays are merged again and printed.

<br/>
<br/>
```raku
sub MAIN( *@nums
	  where { @nums.elems == @nums.grep( { $_ ~~ Int && $_ > 0 } ).elems
				  && @nums.elems > 2 } ) {

    my @array1;
    my @array2;

    @array1.push: @nums.shift;
    @array2.push: @nums.shift;

    while ( @nums ) {
	if ( @array1[ * - 1 ] > @array2[ * - 1 ] ) {
	    @array1.push: @nums.shift;
	}
	else {
	    @array2.push: @nums.shift;
	}
    }


    ( @array1, @array2 ).flat.join( ', ' ).say;

}

```
<br/>
<br/>


# PL/Perl Implementations

Same implementation as in Raku.

<a name="task1plperl"></a>
## PWC 269 - Task 1 - PL/Perl Implementation

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc269.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $nums ) = @_;

   for my $i ( 0 .. $nums->@* - 1 ) {
       for my $j ( $i + 1 .. $nums->@* -1 ) {
       	   my $result = $nums->@[ $i ] | $nums->@[ $j ];
	   if ( $result % 2 == 0 ) {
	      return_next( $nums->@[ $i ] );
	      return_next( $nums->@[ $j ] );
	   }
       }
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 269 - Task 2 - PL/Perl Implementation

Similar implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc269.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $nums ) = @_;

   my ( @array1, @array2 );
   my @nums = $nums->@*;

   push @array1, shift @nums;
   push @array2, shift @nums;

   while ( @nums ) {

   	 if ( $array1[ $#array1 ] > $array2[ $#array2 ] ) {
	    push @array1, shift @nums;
	 }
	 else {
	   push @array2, shift @nums;
	 }
   }

   return_next( $_ ) for ( @array1, @array2 );
   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 269 - Task 1 - PL/PgSQL Implementation

A single query can solve the problem: joining the array unnested to a table with itself and selecting only values where the ored value is even.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc269.task1_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$

   SELECT a, b
   FROM unnest( nums ) a
       , unnest( nums ) b
   WHERE
      a <> b
      AND
      mod( a | b, 2 ) = 0;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 269 - Task 2 - PL/PgSQL Implementation

The idea and implementation is the same as in PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc269.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	array1 int[];
	array2 int[];
	i int;
BEGIN

	i := 1;
	array1 := array_append( array1, nums[ 1 ] );
	array2 := array_append( array2, nums[ 2 ] );
	i := i + 2;

	WHILE i < array_length( nums, 1 ) LOOP
	      IF array1[ i - 2 ] > array2[ i - 1 ] THEN
	      	 array1 := array_append( array1, nums[ i ] );
	      ELSE
	        array2 := array_append( array2, nums[ i ] );
	      END IF;

	      I := I + 2;
	END LOOP;

	RETURN QUERY SELECT v
	       	     FROM unnest( array1 ) v
		     UNION
		     SELECT v
		     FROM unnest( array2 ) v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 269 - Task 1 - PostgreSQL PL/Java Implementation

A nested loop approach.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc269",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task1_pljava( int[] nums ) throws SQLException {

		List<Integer> couples = new LinkedList<Integer>();

		for ( int i = 0; i < nums.length; i++ )
		    for ( int j = i + 1; j < nums.length; j++ )
			if ( ( nums[ i ] | nums[ j ] ) % 2 == 0 ) {
			    couples.add( nums[ i ] );
			    couples.add( nums[ j ] );
			}

		int result[] = new int[ couples.size() ];
		int i = 0;
		for ( int x : couples ) {
		    result[ i++ ] = x;
		}

		return result;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 269 - Task 2 - PostgreSQL PL/Java Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc269",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task2_pljava( int[] nums ) throws SQLException {
		List<Integer> left = new LinkedList<Integer>();
		List<Integer> right = new LinkedList<Integer>();

		left.add( nums[ 0 ] );
		right.add( nums[ 1 ] );

		for ( int i = 2; i < nums.length; i++ ) {
		    if ( left.get( left.size() - 1 ) > right.get( right.size() - 1 ) )
				left.add( nums[ i++ ] );
		    else
			right.add( nums[ i++ ] );
		}

		int result[] = new int[ left.size() + right.size() ];
		int i = 0;
		for ( int x : left )
		    result[ i++ ] = x;
		for ( int x : right )
		    result[ i++ ] = x;

		return result;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 269 - Task 1 - Python Implementation

As in the other implementations, use a nested loop to scan thru the numbers.

<br/>
<br/>
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    nums = list( map( int, args ) )
    couples = []

    for i in range( 0, len( nums ) ):
        for j in range ( i + 1, len( nums ) ):
            if ( nums[ i ] | nums[ j ] ) % 2 == 0 :
                couples.append( nums[ i ] )
                couples.append( nums[ j ] )

    return couples


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )
```python

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 269 - Task 2 - Python Implementation

Similar, but much more verbose implementation than the Raku one.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    nums = list( map( int, args ) )

    left = []
    right = []

    left.append( nums[ 0 ] )
    right.append( nums[ 1 ] )

    i = 2
    while i < len( nums ):
        if left[ len( left ) - 1 ] > right[ len( right ) - 1 ] :
            left.append( nums[ i ] )
        else:
            right.append( nums[ i ] )
        i += 1

    return ( left, right )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
