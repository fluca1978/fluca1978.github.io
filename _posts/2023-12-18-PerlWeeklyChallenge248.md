---
layout: post
title:  "Perl Weekly Challenge 248: enjoy nested loops!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 248: enjoy nested loops!

This post presents my solutions to the [Perl Weekly Challenge 248](https://perlweeklychallenge.org/blog/perl-weekly-challenge-248/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 248 - Task 1 - Raku](#task1)
- [PWC 248 - Task 2 - Raku](#task2)
- [PWC 248 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 248 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 248 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 248 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 248 - Task 1 in Python](#task1python)
- [PWC 248 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 248 - Task 1 - Raku Implementation

The first task was about finding the distance, in terms of indexes, from a given char in a string of chars. Therefore, given a string and a char, the script must print out how many places (left or right) the given character is distant from the closest char in the string.

<br/>
<br/>
```raku
sub MAIN( Str :$string, Str :$char where { $char.chars == 1 } ) {

    my @distances;
    my @matching;
    my @letters = $string.comb;

    @matching.push: $_ if ( $char eq @letters[ $_ ] ) for 0 ..^ @letters.elems;

    for 0 ..^ @letters.elems -> $me {
		@distances[ $me ] = 0 and next if ( $char ~~ @letters[ $me ] );
		@distances[ $me ] = @matching.map( { abs( $_ - $me ) } ).min;
    }

    @distances.join( ', ' ).say;
}

```
<br/>
<br/>

The implementation first of all searches for all the indexes that contains the given character, and stores them in the `@matching` array.
Then I loop over all the `@letters` of the given string and, in the case is not equal to the searched for character, computes the `min` distance from any of the given indexes. To compute the minimum distance, I `map` the `@matching` indexes with the `abs` value of the indexes itself minus the current letter index.


<a name="task2"></a>
## PWC 248 - Task 2 - Raku Implementation

Given a matrix, compute a matrix where each element is the sum of the `2x2` matrix with the top-leftmost corner placed in the current element.

<br/>
<br/>
```raku
sub MAIN() {
    my $a = [
              [1,  2,  3,  4],
              [5,  6,  7,  8],
              [9, 10, 11, 12]
    ];


    my $b = [];
    for 0 ..^ $a.elems - 1 -> $row {
		$b[ $row ].push: [];
		for 0 ..^ $a[ $row ].elems - 1 -> $col {
		    $b[ $row ][ $col ] = $a[ $row ][ $col ] + $a[ $row ][ $col + 1 ] + $a[ $row + 1 ][ $col ] + $a[ $row + 1 ][ $col + 1 ];
		}
    }

    $b.join( "\n" ).say;
}

```
<br/>
<br/>


I haven't placed code for reading the matrix as input, and to print it out as a well formatted matrix, but it works!

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 248 - Task 1 - PL/Perl Implementation

Same implementation as in Raku, but uses a inner `$min` function to compute the min of a given set of values to do the same `map` trick.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc248.task1_plperl( text, text )
RETURNS TABLE( c char, distance int )
AS $CODE$
   my ( $string, $char ) = @_;

   die "Use a single char!" if ( length( $char ) > 1 );
   die "Not matching at all!" if ( $string !~ / $char /x );


   my $min = sub {
      my $current_min = $_[ 0 ];
      shift;
      for ( @_ ) {
      	  $current_min = $_ if ( $_ < $current_min );
      }

      return $current_min;
   };

   my @distances;
   my @matching;

   my @letters = split //, $string;
   for ( 0 .. @letters - 1 ) {
       next if $letters[ $_ ] ne $char;
       push @matching, $_;
   }


   for my $me ( 0 .. @letters - 1 )  {

       $distances[ $me ] = 0 if ( $letters[ $me ] eq $char );
       $distances[ $me ] = $min->( map { abs( $me - $_ ) } @matching ) if ( $letters[ $me ] ne $char );

       return_next( { c => $letters[ $me ],
     		      distance => $distances[ $me ] } );

   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 248 - Task 2 - PL/Perl Implementation

Same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc248.task2_plperl( int[] )
RETURNS int[]
AS $CODE$
   my ( $matrix ) = @_;

   my $result = [];

   for my $row ( 0 .. $matrix->@* - 1 ) {
       $result->[ $row ] = [];

       for my $col ( 0 .. $matrix->@* - 1 ) {
       	   $result->[ $row ][ $col ] = $matrix->[ $row ][ $col ] + $matrix->[ $row ][ $col + 1 ] + $matrix->[ $row + 1 ][ $col ] + $matrix->[ $row + 1 ][ $col + 1 ];
       }
   }

   return $result;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 248 - Task 1 - PL/PgSQL Implementation

I use a temporary table to store the letters and their indexes, and then use a query to compute the minimum distance.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc248.task1_plpgsql( s text, c char )
RETURNS TABLE ( cc char, distance int )
AS $CODE$
DECLARE
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS distances( cc char, ind int );
	TRUNCATE distances;

	INSERT INTO distances
	SELECT v, row_number() over ()
	FROM regexp_split_to_table( s, '' ) v;


	RETURN QUERY
	SELECT d.cc, ( SELECT min( abs( d2.ind - d.ind ) )
	       	     FROM distances d2
		     WHERE d2.cc = c )
	FROM distances d;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 248 - Task 2 - PL/PgSQL Implementation

Implementation similar to the PL/Perl one. Surprisingly, this implementation is quite short, and comparable in code size to PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc248.task2_plpgsql( matrix int[] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	current_row int[];
BEGIN

	FOR r IN 1 .. array_length( matrix, 1 ) - 1 LOOP

	    current_row = array[ array_length( matrix, 1 ) ];

	    FOR c IN 1 .. array_length( matrix, 2 ) - 1 LOOP
	    	current_row[ c ] = matrix[ r ][ c ] + matrix[ r ][ c + 1 ] + matrix[ r + 1 ][ c ] + matrix[ r + 1 ][ c + 1 ];
	    END LOOP;

	    RETURN NEXT current_row;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 248 - Task 1 - Python Implementation

Implementation similar to the Raku one, with the only difference in the need to `append` data to the array.


<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    string = argv[ 0 ]
    char   = argv[ 1 ]

    matching = []
    for i in range( 0, len( string ) ):
        c = string[ i ]
        if c == char:
            matching.append( i )

    distances = []
    for i in range( 0, len( string ) ):
        if string[ i ] == char:
            distances.append( 0 )
        else:
            distances.append( min( map( lambda x: abs( i - x ), matching ) ) )

    print( ', '.join( map( str, distances ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )


```
<br/>
<br/>



<a name="task2python"></a>
## PWC 248 - Task 2 - Python Implementation

Implementation that reflects the Raku one.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    matrix = argv
    output = []

    for r in range( 0, len( matrix ) - 1 ):

        output.append( [] )

        for c in range( 0, len( matrix[ r ] ) - 1 ):
            output[ r ].append( matrix[ r ][ c ] + matrix[ r ][ c + 1 ] + matrix[ r + 1 ][ c ] + matrix[ r + 1 ][ c + 1 ] )

    print( "\n".join( map( str, output ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    matrix = [
        [1, 0, 0, 0],
        [0, 1, 0, 0],
        [0, 0, 1, 0],
        [0, 0, 0, 1]
    ]

    matrix = [
              [1,  2,  3,  4],
              [5,  6,  7,  8],
              [9, 10, 11, 12]
            ]

    main( matrix )

```
<br/>
<br/>
