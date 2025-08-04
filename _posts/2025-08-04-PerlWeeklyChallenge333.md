---
layout: post
title:  "Perl Weekly Challenge 333: streaming numbers"
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

# Perl Weekly Challenge 333: streaming numbers

This post presents my solutions to the [Perl Weekly Challenge 333](https://perlweeklychallenge.org/blog/perl-weekly-challenge-333/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 333 - Task 1 - Raku](#task1)
- [PWC 333 - Task 2 - Raku](#task2)
- [PWC 333 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 333 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 333 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 333 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 333 - Task 1 in PL/Java](#task1pljava)
- [PWC 333 - Task 2 in PL/Java](#task2pljava)
- [PWC 333 - Task 1 in Python](#task1python)
- [PWC 333 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 333 - Task 1 - Raku Implementation

The first task was, given an array of coordinates (two dimensions) to return if the coordinates are all on the same line.

<br/>
<br/>
```raku
sub MAIN( *@coordinates
	  where { @coordinates.grep( * ~~ Int ).elems == @coordinates.elems && @coordinates.elems %% 2  } ) {

    my @coords;
    for @coordinates -> $x, $y {
		@coords.push: { x => $x, y => $y };
    }

    my $first  = @coords[ 0 ];
    my $second = @coords[ 1 ];

    for 2 ..^ @coords.elems {
	my $current = @coords[ $_ ];
		'False'.say and exit if ( ( $current<x> - $first<x> ) * ( $first<y> - $second<y> )
				  != ( $current<y> - $first<y> ) * ( $first<x> - $second<x> ) );
    }

    'True'.say;
}

```
<br/>
<br/>

The idea is to create an array of coordinates with a pair of `x` and `y`, just for making it easier to access the data.
Then I iterate over all the points to see if every point is within the same line of the first and second points, and fail at the first point that is not.


<a name="task2"></a>
## PWC 333 - Task 2 - Raku Implementation

Given a list of numbers, duplicate every zero trimming the array to its length.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my @result;

    for @nums {
		@result.push: $_;
		@result.push: $_ if ( $_ == 0 );
    }

    @result[ 0 .. @nums.elems - 1 ].join( ', ' ).say;
}

```
<br/>
<br/>


I simply duplicate every zero, and then trim the array to the original size.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 333 - Task 1 - PL/Perl Implementation

Same implementation as in Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc333.task1_plperl( int[] )
RETURNS boolean
AS $CODE$

   my ( $coordinates ) = @_;
   die "Wrong arguments!" if ( $coordinates->@* % 2 != 0 );

   my @coords;
   while ( $coordinates->@* ) {
   	 push @coords, { x => shift( $coordinates->@* ),
	                 y => shift( $coordinates->@* ) };
   }

   my $first  = $coords[ 0 ];
   my $second = $coords[ 1 ];

   for my $i ( 2 .. $#coords ) {
       my $current = $coords[ $i ];
       return 0 if ( ( $current->{ x } - $first->{ x } ) * ( $first->{ y } - $second->{ y } )
                     != ( $current->{ y } - $first->{ y } ) * ( $first->{ x } - $second->{ x } ) );

   }

   return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 333 - Task 2 - PL/Perl Implementation

This time, I keep the limit to output the array.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc333.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $numbers ) = @_;

   my $limit = scalar( $numbers->@* );
   my $index = 0;

   while ( $limit ) {
   	 my $current = $numbers->@[ $index++ ];
	 return_next( $current );
         $limit--;
	 last if ( ! $limit );
	 return_next( $current ) if ( ! $current );
	 $limit-- if ( ! $current );

   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 333 - Task 1 - PL/PgSQL Implementation

I cheat here and use PL/Perl to return the result.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc333.task1_plpgsql( c int[] )
RETURNS boolean
AS $CODE$
   SELECT pwc333.task1_plperl( c );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 333 - Task 2 - PL/PgSQL Implementation

Same implementation as in PL/Perl.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc333.task2_plpgsql( n int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	v int;
	l int;
BEGIN

	l := array_length( n, 1 );

	FOREACH v IN ARRAY n LOOP
		IF l = 0 THEN
		   RETURN;
		END IF;


		RETURN NEXT v;
		l := l - 1;

		IF l = 0 THEN
		   RETURN;
		END IF;

		IF v = 0 THEN
		   RETURN NEXT v;
		   l := l - 1;
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
## PWC 333 - Task 1 - PostgreSQL PL/Java Implementation

A simple implementation that resembles the PL/Perl one.

<br/>
<br/>
```java
    public static final boolean task1_pljava( int coordinates[] ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc333.task1_pljava" );

		List<Map<String, Integer>> coords = new LinkedList<Map<String, Integer>>();

		for ( int i = 0; i < coordinates.length - 1; i++ ) {
		    Map<String, Integer> current = new HashMap<String, Integer>();
		    current.put( "x", coordinates[ i++ ] );
		    current.put( "y", coordinates[ i ] );
		    coords.add( current );
		}

		Map<String, Integer> first  = coords.get( 0 );
		Map<String, Integer> second = coords.get( 1 );

		for ( int i = 2; i < coords.size(); i++ ) {
		    Map<String, Integer> current = coords.get( i );

		    if ( ( current.get( "x" ) - first.get( "x" ) ) * ( first.get( "y" ) - second.get( "y" ) )
			 != ( current.get( "y" ) - first.get( "y" ) ) * ( first.get( "x" ) - second.get( "x" ) ) )
			return false;
		}

		return true;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 333 - Task 2 - PostgreSQL PL/Java Implementation

Similar to the PL/Perl implementation.

<br/>
<br/>
```java
    public static final int[] task2_pljava( int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc333.task2_pljava" );

		List<Integer> result = new LinkedList<Integer>();
		int limit = numbers.length;

		for ( int i = 0; i < numbers.length && limit > 0; i++ ) {
		    result.add( numbers[ i ] );
		    limit--;
		    if ( limit == 0 )
				break;

		    if ( numbers[ i ] == 0 ) {
				result.add( numbers[ i ] );
				limit--;
		    }
		}

		int[] tmp = new int[ result.size() ];
		for ( int i = 0; i < tmp.length; i++ )
		    tmp[ i ] = result.get( i );

		return tmp;

    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 333 - Task 1 - Python Implementation

Same implementation as in PL/Perl.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    coords = []
    for i in range( 0, len( args ) - 1 ):
        if i % 2 != 0:
            continue

        x = int( args[ i ] )
        y = int( args[ i + 1 ] )

        current = {}
        current[ 'x' ] = x
        current[ 'y' ] = y

        coords.append( current )

    first  = coords[ 0 ]
    second = coords[ 1 ]


    for i in range( 2, len( coords ) ):
        current = coords[ i ]
        if ( ( current[ 'x' ] - first[ 'x' ] ) * ( first[ 'y' ] - second[ 'y' ] ) ) != ( ( current[ 'y' ] - second[ 'y' ] ) * ( first[ 'x' ] - second[ 'x' ] ) ):
            return False

    return True



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 333 - Task 2 - Python Implementation

Same implementation as in PL/Perl.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    result = []
    limit  = len( args )
    index  = 0

    while limit > 0:
        current = int( args[ index ] )
        index   = index + 1
        result.append( current )
        limit = limit - 1
        if limit == 0:
            break

        if current == 0:
            result.append( current )
            limit = limit - 1
            if limit == 0:
                break

    return result

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
