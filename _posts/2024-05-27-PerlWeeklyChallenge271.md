---
layout: post
title:  "Perl Weekly Challenge 271: Riff Raff (i.e., the first PWC after the AC/DC live concert!)"
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

# Perl Weekly Challenge 271: Riff Raff (i.e., the first PWC after the AC/DC live concert!)

This post presents my solutions to the [Perl Weekly Challenge 271](https://perlweeklychallenge.org/blog/perl-weekly-challenge-271/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 271 - Task 1 - Raku](#task1)
- [PWC 271 - Task 2 - Raku](#task2)
- [PWC 271 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 271 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 271 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 271 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 271 - Task 1 in PL/Java](#task1pljava)
- [PWC 271 - Task 2 in PL/Java](#task2pljava)
- [PWC 271 - Task 1 in Python](#task1python)
- [PWC 271 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 271 - Task 1 - Raku Implementation

The first task was to find the lowest index row in a binary matrix that contains the max number of `1`.

<br/>
<br/>
```rak
usub MAIN() {

    my $matrix = [ [0, 0],
                   [1, 1],
                   [0, 0],
                 ];

    my $row = 1;
    $matrix.map( { [ $row++, $_.grep( * ~~ 1 ).elems  ] } ).grep( { $_[ 1 ] > 0 } ).map( { $_[0] } ).min.say;
}

```
<br/>
<br/>

I first `map` the `$matrix` to create an array where the first element is the `$row` number, and the second is the counting of `1`.
The result is then `grep`-ped to find only rows that have `1`, and remapped to a list of number of row (indexes), extract the `min` and print out.



<a name="task2"></a>
## PWC 271 - Task 2 - Raku Implementation

The second task is about to sort an array of integers by means of the number of `1` in the binary representation of the number itself.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my %bits;
    # convert numbers into an hash keyed by the binary representation
    %bits{ $_.base( 2 ).comb.grep( * ~~ 1 ).elems }.push: $_ for @nums;

    my @output;
    @output.push: |%bits{ $_ }.sort for %bits.keys.sort;
    @output.say;
}

```
<br/>
<br/>

I build an `%bits` hash that is keyed by the counting of the `1` in the binary representation of the number itself, and that contains arrays of integer values. Then I build a flat list `@output` using the sorted keys and the values, and print it.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 271 - Task 1 - PL/Perl Implementation

Implementation conceptually similar to the Raku one.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc271.task1_plperl( int[][] )
RETURNS int
AS $CODE$

   my ( $matrix ) = @_;
   my $row = 1;

   my @ones =
	   map { $_->[ 0 ] }
	   grep { $_->[ 1 ] > 0 }
	   map { [ $row++,
	           scalar grep { $_ == 1 } $_->@*  # columns
		 ] } $matrix->@*; # rows

   my $min = sub {
      my $min = undef;
      for ( @_ ) {
      	  $min = $_ if ( ! $min || $min > $_ );
      }

      return $min;
   };

   return $min->( @ones );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `@ones` array is built on a complex chain of `map`, `grep` and array dereferencing.
The `$min` inner function is used to compute the min value.



<a name="task2plperl"></a>
## PWC 271 - Task 2 - PL/Perl Implementation

Implementation that mimics the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc271.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $nums ) = @_;


   my $binary = {};

   for ( $nums->@* ) {
       my $ones = scalar
       	  	grep { $_ eq '1' }
		split //, sprintf( "%b", $_ );
       push $binary->{ $ones }->@* , $_;
   }

   for my $key ( sort keys $binary->%* ) {
       return_next( $_ )  for  ( sort $binary->%{ $key }->@* );
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 271 - Task 1 - PL/PgSQL Implementation

This implementation uses a nested loop to traverse the matrix.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc271.task1_plpgsql( matrix int[][] )
RETURNS int
AS $CODE$
DECLARE

	counting int := 0;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS ones( r int, v int );
	TRUNCATE ones;

	FOR r IN 1 .. array_length( matrix, 1 ) LOOP
	    counting := 0;

	    FOR c IN 1 .. array_length( matrix, 2 ) LOOP
	    	IF matrix[ r ][ c ] = 1 THEN
		   counting := counting + 1;
		END IF;
	    END LOOP;

	    INSERT INTO ones
	    VALUES( r, counting );
	END LOOP;

	SELECT r
	INTO counting
	FROM ones
	WHERE v = ( SELECT max( v ) FROM ones )
	ORDER BY r asc
	LIMIT 1;


	RETURN counting;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

With a temporary table, I store the row number and the counting of the `1` in such row, then with a single query I return the first tuple that match the request.


<a name="task2plpgsql"></a>
## PWC 271 - Task 2 - PL/PgSQL Implementation

I use a temporay table to store the binary counting of `1` and the number.
Then, returning the table sorted appropriately solves the issue.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc271.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	current_number int;
	binary_number bit(8);
	ones int;
BEGIN
	CREATE TABLE IF NOT EXISTS binary_table( i int, v int );
	TRUNCATE binary_table;

	FOREACH current_number IN ARRAY nums LOOP
		binary_number := current_number::bit( 8 );

		SELECT count(*)
		INTO ones
		FROM regexp_split_to_table( binary_number::text, '' ) v
		WHERE v::int = 1;

		INSERT INTO binary_table
		VALUES( ones, current_number );
	END LOOP;

	RETURN QUERY SELECT v
	FROM binary_table
	ORDER BY i ASC, v ASC;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 271 - Task 1 - PostgreSQL PL/Java Implementation

I use the stream API to mimic the Perl/Raku implementations.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc271",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int size, int[] matrix ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc271.task1_pljava" );

		Map<Integer, Integer> ones = new HashMap<Integer, Integer>();

		IntStream.range( 0, matrix.length )
		    .forEach( i -> {
			    if ( matrix[ i ] == 1 ) {
					int current_row = i / size;
					ones.putIfAbsent( current_row, 0 );
					ones.put( current_row, ones.get( current_row ) + 1 );
			    }
			} );

		return Collections.min( ones.keySet() );
    }
}

```
<br/>
<br/>


I use an `IntStream` to loop over all the indexes of the matrix, considering that `size` indicates the row length.
For every index, I increment the value in the `ones` map, that is then used to find out the min value of the key (i.e., the rows).


<a name="task2pljava"></a>
## PWC 271 - Task 2 - PostgreSQL PL/Java Implementation

Again, a stream API implementation.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc271",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task2_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc271.task2_pljava" );

		final Map<String, List<Integer> > binary = new HashMap<String, List<Integer> >();

		Arrays.stream( nums )
		    .forEach( v -> {
			    String bin = Integer.toBinaryString( v );
			    int ones   = bin.split( "1" ).length;
			    binary.putIfAbsent( bin, new LinkedList<Integer>() );
			    binary.get( bin ).add( v );
			} );

		int result[] = new int[ binary.values().size() ];
		int index = 0;
		List<String> keys = new LinkedList<String>( binary.keySet() );
		Collections.sort( keys );
		for ( String bin : keys ) {
		    List<Integer> values = binary.get( bin );
		    Collections.sort( values );
		    for ( int v : values )
			result[ index++ ] = v;
		}

		return result;
    }
}

```
<br/>
<br/>


I convert the array into a stream, and iterate over all the values adding them to the list keyed by the number of ones in the binary representation.
Then, a lot of code to order the map and the inner lists and return the flat array.


# Python Implementations

<a name="task1python"></a>
## PWC 271 - Task 1 - Python Implementation

Similar to the other implementations, I use the special char `|` to indicate the end of a matrix row.
Therefore, the first part of the code is used to built the matrix out of the command line.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    matrix = []
    row    = 0
    matrix.append( [] )
    for v in args:
        if v == '|':
            row += 1
            matrix.append( [] )
            continue

        matrix[ row ].append( int( v ) )

    ones = {}
    for r in range( 0, len( matrix ) ):
        counting = len( list( filter( lambda x: x == 1, matrix[ r ] ) ) )
        ones[ str( r ) ] = counting

    min_row = None
    for r in ones:
        if ones[ r ] > 0:
            if min_row is None or min_row > int( r ):
                min_row = int( r ) + 1

    return min_row





# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 271 - Task 2 - Python Implementation

The idea is the same as in the other implementations: use a dictioary to store the list of numbers for every coutning of `1` in the binary representation.
Note that I sort the list of numbers at every iteration, that is not efficient.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    binary = {}

    for v in args:
        b = "{0:08b}".format( int( v ) )
        c = 0
        for o in b:
            if o == '1':
                c += 1

        if not c in binary:
            binary[ c ] = []
        binary[ c ].append( int( v ) )
        binary[ c ] = list( sorted( binary[ c ] ) )

    binary = dict( sorted( binary.items() ) )
    result = []
    for b in binary:
        for v in binary[ b ]:
            result.append( v )

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
