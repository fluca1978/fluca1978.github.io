---
layout: post
title:  "Perl Weekly Challenge 251: rows and cols, what a mess!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 251: rows and cols, what a mess!

This post presents my solutions to the [Perl Weekly Challenge 251](https://perlweeklychallenge.org/blog/perl-weekly-challenge-251/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 251 - Task 1 - Raku](#task1)
- [PWC 251 - Task 2 - Raku](#task2)
- [PWC 251 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 251 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 251 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 251 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 251 - Task 1 in Python](#task1python)
- [PWC 251 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 251 - Task 1 - Raku Implementation

The first task was about computing the sum of the string concatenation of a list of integers, keeping two integers at the time.
This is quite straightforward in Raku, since you can *consume* two integers within the same loop.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems and @nums.elems %% 2 } ) {
    my $sum = 0;
    for @nums -> $l, $r {
		$sum += $l ~ $r;
    }

    $sum.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 251 - Task 2 - Raku Implementation

The second task was about finding the number, if any, in a matrix, that is at the same time the min value for its row and the max value for its column.

<br/>
<br/>
```raku
sub MAIN() {

    my $matrix = [ [ 3,  7,  8],
                   [ 9, 11, 13],
                   [15, 16, 17] ];

    my @max-col;
    for 0 ..^ $matrix[ 0 ].elems -> $col {
		my @current-col.push: $matrix[ $_ ][ $col ] for 0 ..^ $matrix.elems;
		@max-col[ $col ] = @current-col.max;
    }

    for 0 ..^ $matrix.elems -> $row {
		my $min = Nil;
		my $min-col = Nil;

		for 0 ..^ $matrix[ $row ].elems -> $col {
		    if ( ! $min ||  $matrix[ $row ][ $col ] < $min ) {
				$min-col = $col;
				$min = $matrix[ $row ][ $col ];
		    }
		}

		$min.say and exit if ( @max-col[ $min-col ] == $min );
    }

    '-1'.say;
}

```
<br/>
<br/>

First of all I scan the matrix by means of every column, searching for every column the max value and storing it into the `@max-col` array so that the array is indexed by the column.
Then I traverse the matrix by row and column, and search for every row the min value keeping track of the index at which the min value is found. Then, if the min value for the current row is the same as the max value for its column, I found a match and terminate.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 251 - Task 1 - PL/Perl Implementation

Quite simple, even if Perl does not allow to consume two elements at the same time, so I simply skip every odd element in the array indexing.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc251.task1_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $nums ) = @_;
   my $sum = 0;

   for  ( 0 .. $nums->@* - 2 ) {
	next if $_ % 2 != 0;
        $sum += $nums->@[ $_ ] . $nums->@[ $_ + 1 ];
   }

   return $sum;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 251 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, I keep track of the max of every column and compare it with the column index of the min value for every row to seek for a match.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc251.task2_plperl( int[][] )
RETURNS int
AS $CODE$
   my ( $matrix ) = @_;

   my $max_col = undef;

   # search the max column value
   my @max_col_indexes;
   for my $col ( 0 .. $matrix->@[ 0 ]->@* - 1 ) {
       $max_col_index[ $col ] = undef;

       for my $row ( 0 .. $matrix->@* - 1 ) {
       	   $max_col_index[ $col ] = $matrix->@[ $row ]->[ $col ] if ( ! $max_col_index[ $col ] || $max_col_index[ $col ] < $matrix->@[ $row ]->[ $col ] );
       }
   }
   for my $row ( 0 .. $matrix->@* - 1 ) {
       for my $col ( 0 .. $matrix->@[ $row ]->@* - 1 ) {
       	   $max_col = $matrix->@[ $row ]->[ $col ] if ( ! $max_col || $max_col < $matrix->@[ $row ]->[ $col ] );
       }
   }

  for my $row ( 0 .. $matrix->@* - 1 ) {
      my $current_min = undef;
      my $current_min_index = 0;

     for my $col ( 0 .. $matrix->@[ $row ]->@* - 1 ) {
     	 if ( ! $current_min || $matrix->@[ $row ]->[ $col ] < $current_min ) {
	    $current_min = $matrix->@[ $row ]->[ $col ];
	    $current_min_index = $col;
	 }
     }

     if ( $current_min == $max_col_index[ $current_min_index ] ) {
     	return $current_min;
     }


  }

  return -1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 251 - Task 1 - PL/PgSQL Implementation

Similar to the PL/Perl approach, but this time I need to skip even indexed elements since the array in SQL are indexed starting at 1!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc251.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	s text;
	v int := 0;
BEGIN
	FOR i IN 1 .. array_length( nums, 1 ) - 1 LOOP
	    IF i % 2 = 0 THEN
	       CONTINUE;
	    END IF;

	    v := v + ( nums[ i ]::text || nums[ i + 1 ]::text )::int;
	END LOOP;

	RETURN v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 251 - Task 2 - PL/PgSQL Implementation

Here I cheat: I delegate the execution to the PL/Perl function, since dealing with multidimensional arrays in SQL is not that fun.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc251.task2_plpgsql( matrix int[][] )
RETURNS int
AS $CODE$
   SELECT pwc251.task2_plperl( matrix );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 251 - Task 1 - Python Implementation

Very simple implementation: I concatenate two elements as string, convert the result into an int and accumulate.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    sum = 0
    for i in range( 0, len( argv ) - 1 ):
        if i % 2 != 0:
            continue
        sum += int( argv[ i ] + argv[ i + 1 ] )

    return sum


# invoke the main without the command itself
if __name__ == '__main__':
    print( main( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 251 - Task 2 - Python Implementation

A quite long implementation, but essentially the same as in Raku. I first need to convert a single string into a matrix, assuming every row is separated by a `|` sign. Then I seek for the max of every column, storing it as an indexed array, then traverse the matrix and for every min value of every row compare its index with the max value in the stored array of max values: if the values are the same, I found a match.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    matrix = []
    row = 0
    matrix.append( [] )

    for x in argv[ 0 ]:
        if x == ' ':
            continue

        # change row
        if x == '|':
            row += 1
            matrix.append( [] )
            continue

        matrix[ row ].append( int( x ) )

    # make indexes of max columns
    max_cols = []
    for col in range( 0, len( matrix ) ):
        current_max = None
        for row in range( 0, len( matrix[ 0 ] ) ):
            if not current_max or current_max < matrix[ row ][ col ] :
                current_max = matrix[ row ][ col ]

        max_cols.append( current_max )

    for row in range( 0, len( matrix ) ):
        current_min = None
        current_min_col = None

        for col in range( 0, len( matrix[ row ] ) ):
            if not current_min or current_min > matrix[ row ][ col ]:
                current_min = matrix[ row ][ col ]
                current_min_col = col

        if current_min == max_cols[ current_min_col ]:
            return current_min

    return -1




# invoke the main without the command itself
if __name__ == '__main__':
    print( main( sys.argv[ 1: ] ) )

```
<br/>
<br/>
