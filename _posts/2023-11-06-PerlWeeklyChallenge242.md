---
layout: post
title:  "Perl Weekly Challenge 242: grepping the arrays for fun and profit!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 242: grepping the arrays for fun and profit!

This post presents my solutions to the [Perl Weekly Challenge 242](https://perlweeklychallenge.org/blog/perl-weekly-challenge-242/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 242 - Task 1 - Raku](#task1)
- [PWC 242 - Task 2 - Raku](#task2)
- [PWC 242 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 242 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 242 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 242 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 242 - Task 1 in Python](#task1python)
- [PWC 242 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 242 - Task 1 - Raku Implementation

The first task was to take two arrays of integers, and print out which elements of the first array were missing in the second array and vice-versa.

<br/>
<br/>
```raku
sub MAIN( Str $left, Str $right ) {

    my @left = $left.split( ',' ).map( *.Int );
    my @right = $right.split( ',' ).map( *.Int );

    my ( @missing-left, @missing-right );

    @missing-left.push: $_ if ( ! @left.grep( $_ ) )  for @right;
    @missing-right.push: $_ if ( ! @right.grep( $_ ) )  for @left;

    ( @missing-left, @missing-right ).say;
}

```
<br/>
<br/>

Since Raku allows a postfix `for` with an attached `if`, it becomes quite simple to build up the missing arrays when the `grep` fails to find an element of the first array into the second.


<a name="task2"></a>
## PWC 242 - Task 2 - Raku Implementation

The second task was about flipping a matrix given as input: each row has to be reversed from left to right and every binary digit has to be negated.

<br/>
<br/>
```raku
sub MAIN() {

    my @matrix =  [ 1, 1, 0 ],
		  [ 0, 1, 1 ],
		  [ 0, 0, 1 ],
    ;

    my @output;
    for @matrix -> $row {
		my $new-row = $row.join.flip.comb.map( { $_ == 0 ?? 1 !! 0 } );
		@output.push: [ $new-row ];
    }

    @output.join( "\n" ).say;
}

```
<br/>
<br/>

I decided to transform every row of the matrix into a string, to `flip` it, break again into pieces by means of `comb`, and `map` every element negating it. The new row is then placed into an `output` array to be printed out.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 242 - Task 1 - PL/Perl Implementation

The implementation is the same as in the Raku part, it is only a little more verbose since in Perl you cannot have a postfix `for` with an attached `if`.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc242.task1_plperl( int[], int[] )
RETURNS TABLE( left int[], right int[] )
AS $CODE$
   my ( $left, $right ) = @_;

   my $missing_left = [];
   my $missing_right = [];

   for my $current ( $left->@* ) {
       push $missing_right->@*, $current if ( ! grep( { $_ == $current } $right->@* ) );
   }

   for my $current ( $right->@* ) {
       push $missing_left->@*, $current if ( ! grep( { $_ == $current } $left->@* ) );
   }

   return_next( { left => $missing_left,
   		  right => $missing_right } );

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 242 - Task 2 - PL/Perl Implementation

Same approach as the Raku one, where I stringify the matrix row, I flip it and split again to remap to something different.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc242.task2_plperl( int[] )
RETURNS int[]
AS $CODE$
   my ( $matrix ) = @_;

   my $output;


   for my $row ( $matrix->@* ) {
       push $output->@*,
       	    [ map { $_ == 0 ? 1 : 0 } split( //, reverse( join( '', $row->@* ) ) ) ];

   }

   return $output;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 242 - Task 1 - PL/PgSQL Implementation

A single query to the rescue: I append the output of a first query to the second one.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc242.task1_plpgsql( left_array int[], right_array int[] )
RETURNS TABLE( which_array text, v int )
AS $CODE$
	SELECT 'FIRST', la
	FROM  unnest( left_array ) la
	WHERE la NOT IN ( SELECT unnest( right_array ) )

	UNION ALL

	SELECT 'SECOND', ra
	FROM  unnest( right_array ) ra
	WHERE ra NOT IN ( SELECT unnest( left_array ) );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The idea is that every query extracts from an array only the elements that don't match the other array. I use a text column to indicate from where the result comes from.


<a name="task2plpgsql"></a>
## PWC 242 - Task 2 - PL/PgSQL Implementation

In this implementation I iterate over every row of the matrix and build an array of the reversed values to return at each iteration.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc242.task2_plpgsql( matrix int[], l int )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	current_row text;
	current int;
	index int;
	to_return int;
	result int[];
BEGIN
	index := 1;
	current_row := '';

	FOREACH current IN ARRAY matrix LOOP
		current_row := current_row || current::text;
		IF index = l THEN
		   -- the row is now complete, flip and split
		   result := array[]::int[];
		   FOREACH to_return IN ARRAY regexp_split_to_array( reverse( current_row ), '' ) LOOP
		   	   IF to_return = 1 THEN
			      result := array_append( result, 0 );
			   ELSE
			      result := array_append( result, 1 );
			   END IF;
		   END LOOP;

		   RETURN NEXT result;

		   -- start over
		   current_row := '';
		   index := 1;
		ELSE
    		   index := index + 1;
		END IF;
	END LOOP;

	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Note that, since PL/PgSQL does not allow for multidimensional array, I have to split the incoming monodimensional array into a matrix by meaning of an incoming argument `l` that represents the length of the array. I still stringify the initial row content and use `reverse` to flip it, then splitting by means of `regexp_split_to_array`.


# Python Implementations

<a name="task1python"></a>
## PWC 242 - Task 1 - Python Implementation

A really verbose implementation, because of the need to extract the two arrays from the command line arguments.
I assume that every array is divided by a `'|'` on the command line, so the first loop is used only to build the arrays.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    left = []
    right = []
    is_left = True
    for current in argv:
        if current != '|':
            if ( is_left ):
                left.append( int( current ) )
            else:
                right.append( int( current ) )
        else:
            is_left = False

    missing_left = []
    missing_right = []

    for current in left:
        if not current in right:
            missing_left.append( current )

    for current in right:
        if not current in left:
            missing_right.append( current )

    print( ",".join( map( str, missing_left ) ) )
    print( ",".join( map( str, missing_right ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )


```
<br/>
<br/>

Python does not have a `grep` like function, but the `in` operator does what you could expect and checks the presence of an alement in the other array.


<a name="task2python"></a>
## PWC 242 - Task 2 - Python Implementation

Similarly to the previous one, I assume that a `'|'` on the command line arguments means that the matrix row is complete, so in the beginning I loop to build up the matrix multidimensional array,
Then I define a `task2` function that, given an input number, returns the string representing the binary reversed (negated) value. I return a string to make `join` happier later on.

Last, I use `map` with my defined function to remap the `reversed` array representing a single row and print it out.

<br/>
<br/>
```python
import sys



# task implementation
def main( argv ):
    matrix = []
    current_row = 0

    matrix.append( [] )
    for current in argv:
        if current != '|':
            matrix[ current_row ].append( int( current ) )
        else:
            current_row += 1
            matrix.append( [] )

    # inner function to use with map
    # returns a string to make join happy!
    def task2(n):
        if n == 1:
            return "0"
        else:
            return "1"


    for current_row in matrix:
        print( ",".join( map( task2, reversed( current_row ) ) ) )




# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )



```
<br/>
<br/>
