---
layout: post
title:  "Perl Weekly Challenge 258: arrays of nums"
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

# Perl Weekly Challenge 258: arrays of nums

This post presents my solutions to the [Perl Weekly Challenge 258](https://perlweeklychallenge.org/blog/perl-weekly-challenge-258/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 258 - Task 1 - Raku](#task1)
- [PWC 258 - Task 2 - Raku](#task2)
- [PWC 258 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 258 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 258 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 258 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 258 - Task 1 in PL/Java](#task1pljava)
- [PWC 258 - Task 2 in PL/Java](#task2pljava)
- [PWC 258 - Task 1 in Python](#task1python)
- [PWC 258 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 258 - Task 1 - Raku Implementation

The first task was to count how many integers, in a given array, have *even* digits.
This can be solved easily with a single line in Raku:

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    @nums.map( { $_.comb.elems %% 2 ?? 1 !! 0 } ).sum.say;

}

```
<br/>
<br/>

The idea is to `map` the array so that every value is substituted by `1` if the count of digits is even, and then a `sum` of the array is done.


<a name="task2"></a>
## PWC 258 - Task 2 - Raku Implementation

Given an integer and an array of integers, compute the sum of only those integers located at an index which binary representation has the keyed number of `1`s.

<br/>
<br/>
```raku
sub MAIN( Int $k where { $k > 0 } , *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my @indexes = ( 0 ..^ @nums.elems ).grep( { $_.base( 2 ).comb.grep( * == 1 ).elems == $k } );
    @nums[ @indexes ].sum.say;

}

```
<br/>
<br/>

The `@indexes` array is computed by `grep`ping the indexes of the input array so that only those indexes that, converted in `base( 2 )` (binary) have a number of ones equal to the specified amount `$k`. Then the array is sliced and summed.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 258 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, but without having a `sum` function, I iterate on the result of mapping the values into ones.
<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc258.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $numbers ) = @_;

   my @counting = map { split( //, $_ ) % 2 == 0 ? 1 : 0  } ( $numbers->@*  );
   my $sum = 0;
   $sum += $_  for ( @counting );
   return $sum;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 258 - Task 2 - PL/Perl Implementation

Here I create an utility function `$count_binary_ones()` that is used to simplify the `grep` done on every index of the array. Within such utility function, I convert the input number (the array index) into a binary form, `split` it and count how many digits are `1`.

Having the resulting indexes, it is simple to iterate over the array and sum the content.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc258.task2_plperl( int, int[] )
RETURNS int
AS $CODE$

   my ( $k, $numbers ) = @_;

   my $count_binary_ones = sub {
      my $num = shift;
      my @digits = split //, sprintf( '%b', $num );
      return scalar( grep { $_ == 1 } @digits );
   };

   my @indexes = grep { $count_binary_ones->( $_ ) == $k } ( 0 .. $numbers->@* - 1 );
   my $sum = 0;
   $sum += $numbers->@[ $_ ] for ( @indexes );
   return $sum;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 258 - Task 1 - PL/PgSQL Implementation

A single query can solve the task: the CTE reports every number with the count of digits, and the external query counts how many numbers have an even number of digits.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc258.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$
   WITH q_nums AS (
   	SELECT v, array_length( regexp_split_to_array( v::text, '' ), 1 ) as c
	FROM unnest( nums ) v
	)
	SELECT count( * )
	FROM q_nums
	WHERE c % 2 = 0;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 258 - Task 2 - PL/PgSQL Implementation

Here I need to iterate on every array index (starting from `1`), convert it into a binary string (assuming 8 bits suffice), and `sum` all the digits of the binary representation. If the sum is equal to the input argument, the `v_sum` summary of the array content is updated.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc258.task2_plpgsql( k int, nums int[] )
RETURNS int
AS $CODE$
DECLARE
	v_sum int := 0;
	count_of_ones int := 0;
BEGIN
	FOR i IN 1 .. array_length( nums, 1 ) LOOP
		SELECT sum( v::int )
		INTO count_of_ones
		FROM regexp_split_to_table( i::bit( 8 )::text, '' ) v;

		IF count_of_ones <> k THEN
		   continue;
		END IF;

		v_sum := v_sum + nums[ i ];
	END LOOP;

	RETURN v_sum;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 258 - Task 1 - PostgreSQL PL/Java Implementation

Quite simple to implement in Java: if the `length()` of the string representing the number has an even number of characters, then the element is accounted.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc258",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int nums[] ) throws SQLException {
		logger.log( Level.INFO, "Entering task1_pljava" );

		int count = 0;
		for ( int current : nums )
		    count += String.format( "%d", current ).length() % 2 == 0 ? 1 : 0;

		return count;
    }
}

```
<br/>
<br/>


The same task can be rewritten using teh **stream** API:

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc258",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int nums[] ) throws SQLException {
		logger.log( Level.INFO, "Entering task1_pljava" );

		List<Integer> numbers = new LinkedList<Integer>();
		for ( int current : nums )
		    numbers.add( current );

		int count = numbers.stream().mapToInt( current -> {
			if ( String.format( "%d", current ).length() % 2 == 0 )
			    return 1;
			else
			    return 0;
		    } ).sum();

		return count;
    }
}

```
<br/>
<br/>

In the above, the list of `numbers` is `mapToInt` and the lambda expression uses the `current` variable placeholder, returning `1` or `0` if the value contains an even number of digits, and then `sum` does the trick.



<a name="task2pljava"></a>
## PWC 258 - Task 2 - PostgreSQL PL/Java Implementation

A nested loop to the rescue: I iterate over all the indexes, and then convert it to a string to count all the digits equal to `1`.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc258",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( int k, int[] nums ) throws SQLException {
	logger.log( Level.INFO, "Entering task2_pljava" );

		int sum = 0;
		int count = 0;
		for ( int i = 0; i < nums.length; i++ ) {
		    count = 0;
		    for ( String d : Integer.toBinaryString( i ).split( "" ) )
			if ( d.equals( "1" ) )
			    count++;


		    if ( count == k )
			sum += nums[ i ];
		}

		return sum;
    }
}

```
<br/>
<br/>

The same result can be achieved with the **stream** API:

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc258",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( int k, int[] nums ) throws SQLException {
			logger.log( Level.INFO, "Entering task2_pljava" );

			return IntStream.range( 0, nums.length )
			    .map( current -> {
				    int ones  = 0;
				    for ( String d : Integer.toBinaryString( current ).split( "" ) )
					if ( d.equals( "1" ) )
					    ones ++;


				    if ( ones  == k )
						return nums[ current ];
				    else
						return 0;

				} ).sum();

    }
}

```
<br/>
<br/>

Here the `IntStream`, an utility stream made only of integers, is used to traverse the indexes of the `nums` array, and every item is mapped to the value at which the index points to, in the case the test pass, or zero in the case of failure. Then the `sum` does the trick.


# Python Implementations

<a name="task1python"></a>
## PWC 258 - Task 1 - Python Implementation

Since Python allows for a `len` of a string, it is quite simple to count the length (and hence, if the number of digits is even) of a given input value. This solution mimics the Java one.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    sum = 0
    for n in args:
        if len( n ) % 2 == 0:
            sum += 1

    return sum


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 258 - Task 2 - Python Implementation

A very unreadable solution!

<br/>
<br/>
```python
import sys


# task implementation
# the return value will be printed
def task_2( args ):
    k = int( args[ 0 ] )
    nums = list( map( int, args[ 1: ] ) )

    def is_index_ok( v, k ):
        b = '{0:08b}'.format( v )
        count = sum( map( int, list( '{0:08b}'.format( v ) ) ) )
        return count == k

    indexes = list( filter( lambda i: is_index_ok( i, k ),
                            range(0, len( nums ) ) ) )

    summy = 0
    for i in indexes:
        summy += nums[ i ]

    return summy


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>

I define a `is_index_ok` function that accepts a number, converts it into a binary string (assuming 8 bits suffice), and convert the string into a `list`, then `map`ping into integers and summing it. This produces the count of `1` in the index binary value. Then, the function returns `True` if the count of `1` is equal to the `k` value.

The function is then used as the filter criteria into the `filter`ing of the indexes, so to get the `indexes` array to sum, that are then summed into a `summy` result.
