---
layout: post
title:  "Perl Weekly Challenge 261: a short one!"
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

# Perl Weekly Challenge 261: a short one!

This post presents my solutions to the [Perl Weekly Challenge 261](https://perlweeklychallenge.org/blog/perl-weekly-challenge-261/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 261 - Task 1 - Raku](#task1)
- [PWC 261 - Task 2 - Raku](#task2)
- [PWC 261 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 261 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 261 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 261 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 261 - Task 1 in PL/Java](#task1pljava)
- [PWC 261 - Task 2 in PL/Java](#task2pljava)
- [PWC 261 - Task 1 in Python](#task1python)
- [PWC 261 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 261 - Task 1 - Raku Implementation

The first task was about computing the absolute difference between the sum of a given array of integers and the array of the (repeated) single digits of every number in the array.
This is quite easy to solve thanks to all the methods an array provide.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my $result = @nums.sum -  @nums.map( *.comb ).flat.sort.sum;
    ( $result < 0 ?? $result * -1 !! $result ).say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 261 - Task 2 - Raku Implementation

The second task was to compute the double of a starting number, assuming every time the current value is foundin a given array the double is computed.

<br/>
<br/>
```raku
sub MAIN( Int $start is copy,
	  *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {

    while ( @nums.grep( * ~~ $start ) ) {
		$start *= 2;
    }

    $start.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 261 - Task 1 - PL/Perl Implementation

Using a loop to simultaneously compute the sum of the digits and of the numbers.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc261.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   my ( $sum, $sum_digits ) = ( 0, 0 );

   for my $current ( $nums->@* ) {
       $sum += $current;
       for my $digit ( split //, $current ) {
       	   $sum_digits += $digit;
       }
   }

   my $result = $sum - $sum_digits;
   return $result < 0 ? $result * -1 : $result;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 261 - Task 2 - PL/Perl Implementation

Same approach as in Raku implementation, short and sweet.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc261.task2_plperl( int, int[] )
RETURNS int
AS $CODE$

   my ( $start, $nums ) = @_;

   while ( grep( { $start == $_ } $nums->@* ) ) {
   	 $start *= 2;
   }

   return $start;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 261 - Task 1 - PL/PgSQL Implementation

A single query with a CTE can progressively provide the sum of the digits and of the numbers.
<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc261.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$

WITH sum_numbers AS ( SELECT SUM( n::int ) AS y FROM unnest( nums ) n )
     , digits AS ( SELECT regexp_split_to_table( n::text, '' ) d
       	      	   FROM unnest( nums ) n )
      , sum_digits AS ( SELECT SUM( d::int ) AS x FROM digits d )
     , result AS ( SELECT y - x AS r
                   FROM sum_numbers, sum_digits )

     SELECT CASE WHEN r > 0 THEN r
            ELSE r * -1
	    END

    FROM result;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 261 - Task 2 - PL/PgSQL Implementation

A loop (without conditions) to query the numbers and find out if the current computed value is in the list.
If the special boolean flag `FOUND` is true, the query did succeed, so the value is doubled, otherwise the function stops and returns the current value.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc261.task2_plpgsql( s int, nums int[] )
RETURNS int
AS $CODE$
DECLARE

BEGIN
	LOOP
		PERFORM s
		FROM unnest( nums ) n
		WHERE n::int = s;

		IF FOUND THEN
		   s := s * 2;
		ELSE
		  RETURN s;
		END IF;
	END LOOP;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 261 - Task 1 - PostgreSQL PL/Java Implementation

Similarly to the PL/Perl implementation, at each iteration the function computes the progressive sum of the numbers and of the digits.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc261",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int nums[] ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc261.task1_pljava" );
		int sum_numbers = 0;
		int sum_digits  = 0;

		for ( int current : nums ) {
		    sum_numbers += current;
		    while ( current > 0 ) {
				sum_digits += current % 10;
				current /= 10;
		    }
		}

		return Math.abs( sum_numbers - sum_digits );

    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 261 - Task 2 - PostgreSQL PL/Java Implementation

I implement a very simple `grep` like function to use in a loop, so that the implementation is similar to the PL/PgSQL one.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc261",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( int start, int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc261.task2_pljava" );

		while ( grep( start, nums ) )
		    start *= 2;

		return start;
    }

    private static boolean grep( int needle, int[] nums ) {
		for ( int current : nums )
		    if ( current == needle )
				return true;

		return false;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 261 - Task 1 - Python Implementation

A simple implementation, but requires a nested loop in order to sum the digits.

<br/>
<br/>
```python

def task_1( args ):
    sum_numbers = sum( list( map( int, args ) ) )
    sum_digits  = 0
    for num in args:
        for d in num:
            sum_digits += int( d )

    result = sum_numbers - sum_digits
    if result < 0:
        return result * -1

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 261 - Task 2 - Python Implementation

A very short implementation, based on a `while` than ends when the list provided by `filter` does not have any element.

<br/>
<br/>
```python
def task_2( args ):
    start = int( args[ 0 ] )
    nums  = args[ 1: ]
    while len( list( filter( lambda x: x == str( start ), nums ) ) ) > 0:
        start *= 2

    return start


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
