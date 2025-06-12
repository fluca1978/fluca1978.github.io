---
layout: post
title:  "Perl Weekly Challenge 325: filter and iterate"
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

# Perl Weekly Challenge 325: filter and iterate

This post presents my solutions to the [Perl Weekly Challenge 325](https://perlweeklychallenge.org/blog/perl-weekly-challenge-325/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 325 - Task 1 - Raku](#task1)
- [PWC 325 - Task 2 - Raku](#task2)
- [PWC 325 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 325 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 325 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 325 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 325 - Task 1 in PL/Java](#task1pljava)
- [PWC 325 - Task 2 in PL/Java](#task2pljava)
- [PWC 325 - Task 1 in Python](#task1python)
- [PWC 325 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 325 - Task 1 - Raku Implementation

Given an array of bits, find out the longest set of consecutie ones.

<br/>
<br/>
```raku
sub MAIN( *@bits where { @bits.grep( * ~~ / ^ <[01]>+ $ / ).elems == @bits.elems  } ) {

    ( @bits.join ~~ m:g/ 1+ / ).map( *.chars ).max.say;
}

```
<br/>
<br/>

A single line does the trick: I `join` the array and match globally against a `1` regular expression, mapping the resulting array of strings to their length, finding out the `max` of the list and printing it.


<a name="task2"></a>
## PWC 325 - Task 2 - Raku Implementation

Given a list of numerical prices, print out the discounted items assuming every item is subtracted by the first item with a lower price.

<br/>
<br/>
```raku
sub MAIN( *@prices where { @prices.grep( * ~~ Int ).elems == @prices.elems } ) {
    my @final-prices;
    for 0 ..^ @prices.elems -> $index {
		@final-prices.push: @prices[ $index ] - ( @prices[ $index + 1 .. * ].grep( * < @prices[ $index ] )[ 0 ] // 0 );
    }

    @final-prices.join( ', ' ).say;
}

```
<br/>
<br/>

I build an array `@final-prices` where I place the current `@prices[ $index ]` subtracted of either a zero or the first item found in the remaining array slice that has a value lower than the current one.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 325 - Task 1 - PL/Perl Implementation

Same implementation as in Raku, but giving a text string as input.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc325.task1_plperl( text )
RETURNS int
AS $CODE$

   my ( $bits ) = @_;

   my @found = ( $bits =~ / 1+ /xg );
   return ( sort
           map { length( $_ ) } @found )[ -1 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

To get the longest value, I `sort` the array of `1`s that has been remapped to the length of the strings. I then extract the last element, that has the biggest value.


<a name="task2plperl"></a>
## PWC 325 - Task 2 - PL/Perl Implementation

Same implementation as in Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc325.task2_plperl( int[] )
RETURNS int[]
AS $CODE$

   my ( $prices ) = @_;
   my @final_prices;

   for my $index ( 0 .. $prices->@* - 1 ) {
       push @final_prices, $prices->@[ $index ]
                           - ( ( grep { $_ < $prices->@[ $index ] } $prices->@[ $index + 1 .. $prices->@* - 1 ] )[ 0 ] // 0 );

   }

   return [ @final_prices ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 325 - Task 1 - PL/PgSQL Implementation

A single query can do the trick: I use `regexp_matches` to extract the list of strings containing only `1`s, then extract the `max` of the `length` of every element. I use `-2` for length because `regexp_matches` provides a string as an array with the curly braces (e.g., `{111}`).


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc325.task1_plpgsql( b text )
RETURNS int
AS $CODE$
   SELECT max( length( v::text ) - 2 )
   FROM  regexp_matches( b, '1+', 'g' ) v;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 325 - Task 2 - PL/PgSQL Implementation

An iterative approach where I iterate over all the elements of the array and extract the first value lower than the current one by measn of a single query.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc325.task2_plpgsql( prices int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	current_price int;
BEGIN
	FOR i IN 1 .. array_length( prices, 1 ) LOOP
	    SELECT prices[ i ] -
	    	   ( SELECT v
		     FROM unnest( prices[ i + 1 : array_length( prices, 1 ) - 1 ] ) v
		     WHERE v::int < prices[ i ]
		     LIMIT 1
		  )
           INTO current_price;

	   IF NOT FOUND THEN
	      RETURN NEXT prices[ i ];
	   ELSE
	     RETURN NEXT current_price;
	   END IF;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 325 - Task 1 - PostgreSQL PL/Java Implementation

I use a slightly different approach here: I keep a `current_max` of `1`s and `max_ones` that keep track of the longest set of ones I find.


<br/>
<br/>
```java
    public static final int task1_pljava( String bits ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc325.task1_pljava" );

		int max_ones    = 0;
		int current_max = 0;

		for ( String d : bits.split( "" ) ) {
		    if ( d.equals( "1" ) )
				current_max++;
		    else {
				if ( current_max > max_ones )
					max_ones = current_max;

	            current_max = 0;

		    }
		}

		return max_ones;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 325 - Task 2 - PostgreSQL PL/Java Implementation

Iterative approach with a simple nested loop to find out the values.

<br/>
<br/>
```java
    public static final int[] task2_pljava( int[] prices ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc325.task2_pljava" );

		List<Integer> new_prices = new LinkedList<Integer>();

		for ( int i = 0; i < prices.length; i++ ) {
		    int current = prices[ i ];

		    for ( int j = i + 1; j < prices.length; j++ )
				if ( prices[ j ] < prices[ i ] ) {
				    current -= prices[ j ];
				    break;
				}

		    new_prices.add( current );
		}

		int result[] = new int[ new_prices.size() ];
		for ( int i = 0; i < new_prices.size(); i++ )
		    result[ i ] = new_prices.get( i );

		return result;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 325 - Task 1 - Python Implementation

Same approach as PL/Java.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    bits  = list( map( int, args ) )
    f_max = 0
    c_max = 0

    for d in bits:
        if d == 1:
            c_max += 1
        else:
            if c_max > f_max :
                f_max = c_max

            c_max = 0

    return f_max



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 325 - Task 2 - Python Implementation

Same approach as in PL/Java.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    prices     = list( map( int, args ) )
    new_prices = []

    for i in range( 0, len( prices ) ) :
        current  = prices[ i ]
        next_min = list( filter( lambda x: x < current, prices[ i + 1 : ] ) )
        if len( next_min ) > 0 :
            next_min = next_min[ 0 ]
        else:
            next_min = 0

        new_prices.append( current - next_min )

    return new_prices



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
