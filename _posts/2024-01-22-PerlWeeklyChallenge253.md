---
layout: post
title:  "Perl Weekly Challenge 253: not so easily handling multidimensional arrays"
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

# Perl Weekly Challenge 253: not so easily handling multidimensional arrays

This post presents my solutions to the [Perl Weekly Challenge 253](https://perlweeklychallenge.org/blog/perl-weekly-challenge-253/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 253 - Task 1 - Raku](#task1)
- [PWC 253 - Task 2 - Raku](#task2)
- [PWC 253 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 253 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 253 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 253 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 253 - Task 1 in PL/Java](#task1pljava)
- [PWC 253 - Task 2 in PL/Java](#task2pljava)
- [PWC 253 - Task 1 in Python](#task1python)
- [PWC 253 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 253 - Task 1 - Raku Implementation

Given an array of strings and a separator, print out all the words that can be split from the array using the separator.

<br/>
<br/>
```raku
sub MAIN( $separator, *@words ) {
    @words.split( $separator, :skip-empty ).join( ',' ).say;
}
```
<br/>
<br/>



<a name="task2"></a>
## PWC 253 - Task 2 - Raku Implementation

Given a binary matrix, compute the *weak rows* and print the indexes of the rows from the weakenest to the strongest one.
A row is considered weak if it has less `1`s of another row, and in the case two rows have the same exact number of `1`s, the one with the smallest index is weaker.

<br/>
<br/>
```raku
sub MAIN() {

    my $matrix = [
                   [1, 1, 0, 0, 0],
                   [1, 1, 1, 1, 0],
                   [1, 0, 0, 0, 0],
                   [1, 1, 0, 0, 0],
                   [1, 1, 1, 1, 1]
    ];

    my %ones;
    for 0 ..^ $matrix.elems -> $row_index {
		my $count = $matrix[ $row_index ].grep( * ~~ 1 ).elems;
		%ones{ $count }.push: $row_index;
    }


    my @rows.push: |%ones{ $_ }.sort for %ones.keys.sort;
    @rows.join( ', ').say;
}

```
<br/>
<br/>

The idea is quite simple: I use a `%ones` hash keyed at the count of `1`s in a row, that contains an array with all the indexes of the rows with such cpount. Then I sort the keys, i.e., the counting of `ones`, sort the values in the rows (i.e., the indexes) and print out everything. The flatting of the list is due to the fact the Raku preseves internal lists, in this case arrays of indexes, while I want a flat list to output.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 253 - Task 1 - PL/Perl Implementation

Simple enough implementation: `split` each input array element by means of `$separator` specified as first argument.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc253.task1_plperl( char, text[] )
RETURNS SETOF text
AS $CODE$

   my ( $separator, $words ) = @_;
   for my $word ( $words->@* ) {
       my $pattern = qr/ [$separator] /x;

       if ( $word !~ $pattern ) {
       	  return_next( $word );
	      next;
       }

       return_next( $_ ) for ( split( /$pattern/, $word ) );
   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

There is a trick, however: I need to build a regular expression that contains the `$separator` into the square backets, in order to protect some particular characters (e.g., `.`).



<a name="task2plperl"></a>
## PWC 253 - Task 2 - PL/Perl Implementation

Same approach as in Raku: I first classify the rows into an hash keyed by the count of `1`s, and then sort the keys and return the sorted list of values.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc253.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $matrix ) = @_;

   my $ones = {};

   for my $row_index ( 0 .. $matrix->@* - 1 ) {
       my $count = scalar grep { $_ == 1 } $matrix->[ $row_index ]->@*;
       push $ones->{ $count }->@*, $row_index;
   }

   for my $count ( sort keys $ones->%* ) {
       return_next( $_ ) for ( sort $ones->{ $count }->@* );
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 253 - Task 1 - PL/PgSQL Implementation

For every words in the array of text, I use a query to split the values by means of the separator (again, in square brackets) and return the result set.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc253.task1_plpgsql( s char, words text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	current_word text;
BEGIN
	FOREACH current_word IN ARRAY words LOOP
		RETURN QUERY
		SELECT regexp_split_to_table( current_word, '[' || s || ']' );
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 253 - Task 2 - PL/PgSQL Implementation

Handling multi-dimensional arrays in PL/PgSQL is a mess. Therefore I use a nested loop to examine every element of the matrix, and count the number of ones. I put the results in a table `ones` that is used an a lookup hash, and then return the right row indexes according to the ordering of keys and values

<br/>
<br/>
```java
CREATE OR REPLACE FUNCTION
pwc253.task2_plpgsql( matrix int[][] )
RETURNS SETOF int
AS $CODE$
DECLARE
	current_row int[];
	c_ones int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS ones( r int, c int );
	TRUNCATE ones;

	FOR i IN 1 .. array_length( matrix, 1 ) LOOP
	    c_ones := 0;
	    FOR j IN 1 .. array_length( matrix, 2 ) LOOP
	    	c_ones := c_ones + matrix[ i ][ j ];
	    END LOOP;

    	    INSERT INTO ones
	    SELECT i, c_ones;

	END LOOP;

	RETURN QUERY
	SELECT r
	FROM ones
	ORDER BY c DESC, r ASC;


RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 253 - Task 1 - PostgreSQL PL/Java Implementation

An approach similar to the PL/Perl one: I iterate other all the words, and split them by means of the `split` method using a regular expression that contains the `separator` into square brackets to avoid regular expression interpolation of special characters.


<br/>
<br/>
```java
import org.postgresql.pljava.*;
import org.postgresql.pljava.annotation.Function;
import static org.postgresql.pljava.annotation.Function.Effects.IMMUTABLE;
import static org.postgresql.pljava.annotation.Function.OnNullInput.RETURNS_NULL;

import java.util.*;
import java.util.logging.*;

public class Task1 {
    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
    public static final String[] task1_pljava( String separator, String[] words ) {
		List<String> result = new LinkedList<String>();

		for ( String w : words ) {
		    if ( ! w.contains( separator ) )
				result.add( w );
		    else
				for ( String ww : w.split( "[" + separator + "]" ) ) {
					result.add( ww );
			}
		}

		return result.toArray( new String[ result.size() ] );
    }

}

```
<br/>
<br/>




<a name="task2pljava"></a>
## PWC 253 - Task 2 - PostgreSQL PL/Java Implementation

If handling multi-dimensional arrays in PL/PgSQL is difficult, it is even mor in PL/Java: arrays are monodimensional and there is no information about the dimension. Therefore, the function accepts a `size` value that determines how many elements a single matrix rows has.
The implementation then is the same as PL/Perl: `ones` is an hash keyed at the counting of ones for every row, and which values are the row indexes. Then, it is only a matter of sorting keys and values.

<br/>
<br/>
```java
import org.postgresql.pljava.*;
import org.postgresql.pljava.annotation.Function;
import static org.postgresql.pljava.annotation.Function.Effects.IMMUTABLE;
import static org.postgresql.pljava.annotation.Function.OnNullInput.RETURNS_NULL;

import java.util.*;

public class Task2 {

       @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
       public static final Integer[] task2_pljava( int size, int[] matrix ) throws Exception {
		   int row_index = 0;
		   Map<Integer, List<Integer>> ones = new HashMap<Integer, List<Integer>>();

		   int number_of_ones = 0;

		   for ( int i = 0; i < matrix.length; i++ ) {
		       number_of_ones += matrix[ i ];
		       if ( ( i + 1 ) % size == 0 || i == matrix.length ) {

				   if ( ! ones.containsKey( number_of_ones ) )
				       ones.put( number_of_ones, new LinkedList<Integer>() );

				   ones.get( number_of_ones ).add( row_index++ );
				   number_of_ones = 0;
		       }


		   }

		   List<Integer> keys = new LinkedList<Integer>( ones.keySet() );
		   Collections.sort( keys );

		   List<Integer> result = new LinkedList<Integer>();
		   for ( int k : keys ) {
		       List<Integer> values = ones.get( k );
		       Collections.sort( values );
		       result.addAll( values );
		   }

		   return result.toArray( new Integer[ result.size() ] );
       }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 253 - Task 1 - Python Implementation

Quite simple implementation: I split the words using a regular expression and then join everything I find.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    separator   = args[ 0 ];
    split_words = []
    for i in range( 1, len( args ) ):
        current_words = args[ i ]
        for w in re.split( "[%s]" % separator, current_words ):
            split_words.append( w )

    return ','.join( split_words )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 253 - Task 2 - Python Implementation

The approach is the same as in Raku and PL/Perl: I use a dictionary keyed by the count of `1`s to store an array with the indexes of the matrix rows.
Then I do sort everything and get back to join into a single string.


<br/>
<br/>
```python
import sys
import collections

# task implementation
# the return value will be printed
def task_2( args ):
    ones = {}
    for row_index in range( 0, len( args ) ):
        sum = 0
        for x in args[ row_index ]:
            sum += x

        if not sum in ones:
            ones[ str( sum ) ] = []

        ones[ str( sum ) ].append( row_index )

    keys = list( ones.keys() )
    keys.sort()
    result = ""
    for k in keys:
        ones[ k ].sort()
        for v in ones[ k ]:
            result += str( v ) + ','


    return result



# invoke the main without the command itself
if __name__ == '__main__':
    matrix = [
        [1, 1, 0, 0, 0],
        [1, 1, 1, 1, 0],
        [1, 0, 0, 0, 0],
        [1, 1, 0, 0, 0],
        [1, 1, 1, 1, 1]
        ]
    print( task_2( matrix ) )

```
<br/>
<br/>
