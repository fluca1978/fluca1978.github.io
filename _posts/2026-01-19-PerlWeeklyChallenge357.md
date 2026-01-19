---
layout: post
title:  "Perl Weekly Challenge 357: arrays everywhere!"
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

# Perl Weekly Challenge 357: arrays everywhere!

This post presents my solutions to the [Perl Weekly Challenge 357](https://perlweeklychallenge.org/blog/perl-weekly-challenge-357/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 357 - Task 1 - Raku](#task1)
- [PWC 357 - Task 2 - Raku](#task2)
- [PWC 357 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 357 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 357 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 357 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 357 - Task 1 in PL/Java](#task1pljava)
- [PWC 357 - Task 2 in PL/Java](#task2pljava)
- [PWC 357 - Task 1 in Python](#task1python)
- [PWC 357 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 357 - Task 1 - Raku Implementation

The first task was about to compute the required iterations to transform a given input number into the Kaprekar constant.

<br/>
<br/>
```raku
sub MAIN( Int $value is copy
	  where { $value < 10000 } ) {
    my $kaprekar = 6174;
    my $iterations = 0;

    while ( $value != $kaprekar ) {
		$value = $value.comb.sort( { $^b <=> $^a } ).join.Int - $value.comb.sort.join.Int;
		$iterations++;
    }

    $iterations.say;
}

```
<br/>
<br/>


The idea is quite simple: iterate unless the numbers are the same, and on each iteration subtract the number obtained by the digits in a descending order from the one obtained in ascending order.


<a name="task2"></a>
## PWC 357 - Task 2 - Raku Implementation

Given an input integer, compute all available fractions that can be made by the numbers from `1` to the given one (included), keeping the smallest numerator ones in case of duplicated decimal value.


<br/>
<br/>
```raku
sub MAIN( Int $numerator where { $numerator > 1 } ) {

    my @nums = 1 .. $numerator [X] 1 .. $numerator;
    my %values;
    for @nums {
		my ( $n, $d ) = $_[ 0 ], $_[ 1 ];
		%values{ $n / $d }.push: "$n/$d";
    }

    %values{ $_ }.sort()[ 0 ].say for %values.keys.sort;
}

```
<br/>
<br/>


The idea is to compute all the cross-join of the list of available numbers, then to use the `%values` hash to store the fraction decimal value and its found representatives.
Then, print out the first (sorted) textual fraction for each decimal value.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 357 - Task 1 - PL/Perl Implementation

The same implementation as in Raku, only a little more verbose.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc357.task1_plperl( int )
RETURNS int
AS $CODE$

   my ( $num ) = @_;
   my $kaprekar = 6174;
   my $iterations = 0;

   while ( $num != $kaprekar ) {
   	 my @digits = split //, $num;
	 $num = join( '', sort( { $b <=> $a } @digits ) ) -
	        join( '', sort( { $a <=> $b } @digits ) );
	$iterations++;
   }

   return $iterations;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 357 - Task 2 - PL/Perl Implementation

Here I use the `Set::CrossProduct` library to get the cross join in a single line, but this requires the code to execute with higher privileges in PostgreSQL.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc357.task2_plperl( int )
RETURNS SETOF text
AS $CODE$

   use Set::CrossProduct;

   my ( $num ) = @_;
   my $fractions = {};
   my $engine = Set::CrossProduct->new( [ [ 1 .. $num ], [ 1 .. $num ] ] );
   while ( my $tuple = $engine->get ) {
   	 my ( $n, $d ) = ( $tuple->[ 0 ], $tuple->[ 1 ] );
	 push $fractions->{ $n / $d }->@*, "$n/$d";
   }


   for my $key ( sort keys $fractions->%* ) {
       my $value = ( sort( $fractions->{ $key }->@* ) )[ 0 ];
       return_next( $value );
   }

   return undef;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

The implementation is the same as in Raku.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 357 - Task 1 - PL/PgSQL Implementation


Almost a single query to solve the problem, but still there is the need for an iterative approach.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc357.task1_plpgsql( n int )
RETURNS int
AS $CODE$
DECLARE
	k int := 6174;
	i int := 0;
BEGIN
	WHILE n <> k LOOP
	      i := i + 1;

	      with digits_asc as (
	           select v
		   from   regexp_split_to_table( n::text, '' ) v
		   order by v asc
		  ),
		  digits_desc as (
		   select v
		   from   regexp_split_to_table( n::text, '' ) v
		   order by v desc
		  ),
		  a as ( select array_to_string( array_agg( a.v ), '' )::int v
		         from digits_asc a
		       ),
		  d as ( select array_to_string( array_agg( d.v ), '' )::int v
		         from digits_desc d
		       )
             select d.v - a.v
	     into n
	     from d d, a a;

	END LOOP;

	RETURN i;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The main CTE produces the digits from the current value, then sort them in two different directions, and compose the numbers that are then subtracted.


<a name="task2plpgsql"></a>
## PWC 357 - Task 2 - PL/PgSQL Implementation

I use a temporary table as a storage for computed values, keeping the decimal value, the numerator, denominator and textual form of the fraction.
After having computed all values, it does suffice to query them in the right order (by decimal value) keeping only one fraction, the one with the lowest numerator.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc357.task2_plpgsql( num int )
RETURNS SETOF text
AS $CODE$
DECLARE
	r record;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS fractions( n int, d int, f text, v numeric );
	TRUNCATE TABLE fractions;

	FOR r IN SELECT n, d
	       	    FROM generate_series( 1, num ) n,
		         generate_series( 1, num ) d LOOP



		INSERT INTO fractions( n, d, f, v )
		VALUES( r.n, r.d, r.n || '/' || r.d, r.n::numeric / r.d::numeric );

	END LOOP;

	RETURN QUERY SELECT f
	            FROM fractions f1
		    WHERE f1.n = ( select min( f2.n )
		                   from fractions f2
				   where f2.v = f1.v
				 )
	       	    GROUP BY v, f
	       	    ORDER BY v asc;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 357 - Task 1 - PostgreSQL PL/Java Implementation


A very verbose solution.


<br/>
<br/>
```java
    @Function( schema = "pwc357",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int value ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc357.task1_pljava" );

		final int k = 6174;
		int iterations = 0;

		while ( value != k ) {
		    List<Integer> asc  = new LinkedList<Integer>();
		    List<Integer> desc = new LinkedList<Integer>();
		    String current = "" + value;

		    for ( String c : current.split( "" ) ) {
				asc.add( Integer.parseInt( c ) );
				desc.add( Integer.parseInt( c ) );
		    }

		    Collections.sort( asc );
		    Collections.sort( desc, Collections.reverseOrder() );

		    StringBuffer b1 = new StringBuffer();
		    StringBuffer b2 = new StringBuffer();
		    for ( int x : asc )
				b1.append( x );

		    for ( int x : desc )
				b2.append( x );

		    value = Integer.parseInt( b2.toString() ) - Integer.parseInt( b1.toString() );


		    iterations++;
		}

		return iterations;

    }
```
<br/>
<br/>


I keep two list where I store the digits in ascending and descending order.
Then I place the digits into two strings and compute the difference of the integer value. If the value is not the constant, re-iterate.


<a name="task2pljava"></a>
## PWC 357 - Task 2 - PostgreSQL PL/Java Implementation

Another very verbose solution.

<br/>
<br/>
```java
    @Function( schema = "pwc357",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String[] task2_pljava( int value ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc357.task2_pljava" );

		List<Integer[]> perms = new LinkedList<Integer[]>();

		for ( int i = 1; i <= value; i++ ) {
		    for ( int j = 1; j <= value; j++ ) {
				perms.add( new Integer[]{ i, j } );
		    }
		}

		Map<Double, Integer[]> fractions = new HashMap<Double, Integer[]>();

		for ( Integer[] current : perms ) {
		    int n = current[ 0 ];
		    int d = current[ 1 ];
		    double v = n / (double) d;

		    if ( fractions.containsKey( v ) ) {
				if ( fractions.get( v )[ 0 ] > n )
					fractions.put( v, new Integer[]{ n, d } );
	        }
		    else
				fractions.put( v, new Integer[]{ n, d } );
		}


		List<Double> keys = new LinkedList<Double>(  fractions.keySet() );
		Collections.sort( keys );

		String[] results = new String[ keys.size() ];

		for ( int i = 0; i < results.length; i++ ) {
		    Integer[] f = fractions.get( keys.get( i ) );
		    results[ i ] = f[ 0 ] + "/" + f[ 1 ];
		}

		return results;

    }
```
<br/>
<br/>


With a nested loop, I generate all possible permutations.
Then I place every value into the `fractions` map, in a similar way to what done in Perl/Raku solutions.
However, this time I directly store only the single fraction with the lowest numerator in case of the same decimal value.

Last, I create an array with the resulting text representations.


# Python Implementations

<a name="task1python"></a>
## PWC 357 - Task 1 - Python Implementation

The solution is quite simple, even if disassembling and reassembling numbers is pain.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    k = 6174
    num = int( args[ 0 ] )
    iterations = 0

    while num != k :
        digits = [ int( i ) for i in str( num ) ]
        num = int( ''.join( list( map( str, sorted( digits, reverse = True ) ) ) ) ) - int( ''.join( list( map( str, sorted( digits ) ) ) ) )
        iterations += 1

    return iterations


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 357 - Task 2 - Python Implementation

This is the very same solution as in PL/Java.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    num = int( args[ 0 ] )
    perms = []

    for i in range( 1, num + 1 ) :
        for j in range( 1, num + 1 ) :
            perms.append( [ i, j ] )

    fractions = {}
    for current in perms :
        n = current[ 0 ]
        d = current[ 1 ]
        v = str( n / d )

        if ( not v in fractions ) or  ( v in fractions and fractions[ v ][ 0 ] > n ):
            fractions[ v ] = [ n , d ]

    for k in sorted( fractions.keys() ):
        current = fractions[ k ]
        print( '%d/%d' % ( current[ 0 ], current[ 1 ] ) )



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
