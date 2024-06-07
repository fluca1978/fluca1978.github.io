---
layout: post
title:  "Perl Weekly Challenge 267: traversing arrays"
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

# Perl Weekly Challenge 267: traversing arrays

This post presents my solutions to the [Perl Weekly Challenge 267](https://perlweeklychallenge.org/blog/perl-weekly-challenge-267/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 267 - Task 1 - Raku](#task1)
- [PWC 267 - Task 2 - Raku](#task2)
- [PWC 267 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 267 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 267 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 267 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 267 - Task 1 in PL/Java](#task1pljava)
- [PWC 267 - Task 2 in PL/Java](#task2pljava)
- [PWC 267 - Task 1 in Python](#task1python)
- [PWC 267 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 267 - Task 1 - Raku Implementation

The first task was about finding out if the product of a list of integers produced a positive, negative or zero result and print a value accordingly.
This is quite simple: Raku has a reduce operator that can do exactly that. But instead of performing the computation, let's use a different approach and see if the list of integers contains at least one zero, in such case the ending product will be zero too. Otherwise, let's see if the list contains an odd number of negative values, in such case the ending result will be also negative. Last, the final product must be positive.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    '0'.say and exit if ( @nums.grep( * == 0 ) );
    '-1'.say and exit if ( @nums.grep( * < 0 ).elems !%% 2 );
    '1'.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 267 - Task 2 - Raku Implementation

The second task was about splitting a string into different lines depending on scores, represented as pixels.

The input is a string and an array of 26 elements with the weight, in pixels, of every single letter of the alphabet. The script has to compute how many parts (lines) must be done on the initial phrase so that every line is at a max score of 100 pixels.
The end result is to print the number of lines and the weight of the last line.

<br/>
<br/>
```raku
sub MAIN( Str $string, *@widths where { @widths.elems == 26 && @widths.elems == @widths.grep( * ~~ Int ).elems } ) {

    my %pixels;
    my $index = 0;
    %pixels{ $_ } = @widths[ $index++ ] for 'a' .. 'z';


    $index = 0;
    my $lines = 1;
    my $pixels = 0;

    for $string.comb -> $letter {
		if ( $pixels + %pixels{ $letter } > 100 ) {
		    $lines++;
		    $pixels = 0;
		}

		$pixels += %pixels{ $letter };

    }

    ( $lines, $pixels ).join( ', ' ).say;
}

```
<br/>
<br/>

Initially, the script has to map every letter to its weight, than I iterate over all the letters found in the phrase and compute the sum of the weight, taking care than once the `100` threshold is reached the number of lines must be increased.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 267 - Task 1 - PL/Perl Implementation

Same approach as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc267.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   return 0 if ( grep { $_ == 0 } $nums->@* );
   return -1 if ( grep( { $_ < 0 } $nums->@* ) % 2 != 0 );
   return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 267 - Task 2 - PL/Perl Implementation

Similar approach to the Raku one.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc267.task2_plperl( text, int[] )
RETURNS table( line int, width int )
AS $CODE$

   my ( $string, $widths ) = @_;

   die if ( $widths->@* != 26 );

   my $pixels = {};
   my $index = 0;
   $pixels->{ $_ } = $widths->@[ $index++ ] for ( 'a' .. 'z' );

   my $sum = 0;
   my $lines = 1;

   for my $letter ( split //, $string ) {
       if ( $sum + $pixels->{ $letter } > 100 ) {
       	  $sum = 0;
	  $lines++;
       }

       $sum += $pixels->{ $letter };
   }

   return_next( { line => $lines, width => $sum } );
   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The array is returned as a table made by two columns and a single row.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 267 - Task 1 - PL/PgSQL Implementation

Here I decided to compute the final product, because it is simpler than seeking for zeros or negative numbers.
As soon as a zero is detected, the loop ends because the result is already known.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc267.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	i int;
	product int;
BEGIN
	product := 1;

	FOREACH i IN ARRAY nums LOOP
		product := product * i;
		IF product = 0 THEN
		   RETURN 0;
		END IF;
	END LOOP;

	IF product > 0 THEN
	   RETURN 1;
	ELSE
          RETURN -1;
	END IF;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 267 - Task 2 - PL/PgSQL Implementation

The implementation is the same as in PL/Perl one, however I use a table as an hash to classify every letter to its own weight.
Note the *ancient* techniquie of considering a char from the ascii chart table, so `a` is character number 97 and the `b` is the following one and so on.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc267.task2_plpgsql( s text, w int[] )
RETURNS TABLE( line int, width int )
AS $CODE$
DECLARE
	l text;
	n_value int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS pixels( letter char, pixels int );
	TRUNCATE pixels;

	FOR i IN 1 .. array_length( w, 1 ) LOOP
	    INSERT INTO pixels
	    VALUES( chr( 97 + i - 1 ), w[ i ] );
	END LOOP;

	width := 0;
	line  := 1;

	FOR l IN SELECT v FROM regexp_split_to_table( s, '' ) v LOOP
		SELECT pixels
		INTO n_value
		FROM pixels
		WHERE letter = l;

		IF n_value + width > 100 THEN
		   width := 0;
		   line  := line + 1;
		END IF;

		width := width + n_value;
	END LOOP;


	RETURN NEXT;
	RETURN;
END


$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 267 - Task 1 - PostgreSQL PL/Java Implementation

I use the stream API to handle the final product and see then if the product is greater than zero or not.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc267",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc267.task1_pljava" );

		final int product[] = new int[1];
		product[ 0 ] = 1;

		 Arrays.stream( nums )
		     .forEach( current -> { product[ 0 ] *= current; } );

		 if ( product[ 0 ] == 0 )
		     return 0;
		 else
		     return product[ 0 ] > 0 ? 1 : -1;
	    }
}

```
<br/>
<br/>

Note the need to use a monodimensional single-cell array to store the accumulation from within the lambda expression.


<a name="task2pljava"></a>
## PWC 267 - Task 2 - PostgreSQL PL/Java Implementation

Again, use the stream API with some tricks.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc267",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task2_pljava( String phrase, int[] widths ) throws SQLException {


		final Map<String, Integer> pixels = new HashMap<String, Integer>();

		final int[] index = new int[1];
		index[ 0 ] = 0;
		IntStream.rangeClosed( 'a', 'z' )
		    .mapToObj( current  -> (char)  current )
		    .forEach( current -> { pixels.put( "" + current, widths[ index[ 0 ]++ ] ); } );

		 int sum = 0;
		 int lines = 1;

		 for ( String current : phrase.split( "" ) ) {
		     int current_pixels = pixels.get( current );
		     if( sum + current_pixels > 100 ) {
				 lines++;
				 sum = 0;
		     }

		     sum += current_pixels;
		 }

		 return new int[]{ lines, sum };

	    }
}

```
<br/>
<br/>

I use a `rangeClosed` to generate the range of characters, that are then converted into effective characters from `mapToObj` and iterate over with `forEach`.
The rest is the same as in the other implementations.


# Python Implementations

<a name="task1python"></a>
## PWC 267 - Task 1 - Python Implementation

I use `filter` to seek for zeros and negative numbers, so that I can quickly decide what the final product will be without having to compute it.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    nums = list( map( int, args ) )
    zeros = list( filter( lambda x: x == 0, nums ) )
    negatives = list( filter( lambda x: x < 0, nums ) )
    if len( zeros ) > 0:
        return 0
    elif len( negatives ) % 2 != 0:
        return -1
    else:
        return 1


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 267 - Task 2 - Python Implementation

The same implementation as in other cases.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    phrase = args[ 0 ]
    widths = {}

    index = 1
    for letter in 'abcdefghiljkmnopqrstuvwxyz':
        widths[ letter ] = int( args[ index ] )
        index += 1

    lines  = 1
    pixels = 0

    for letter in phrase:
        current_width = widths[ letter ]
        if pixels + current_width > 100:
            pixels = 0
            lines += 1

        pixels += current_width

    return [ lines, pixels ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
