---
layout: post
title:  "Perl Weekly Challenge 211: arrays everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 211: arrays everywhere!

This post presents my solutions to the [Perl Weekly Challenge 211](https://perlweeklychallenge.org/blog/perl-weekly-challenge-211/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 211 - Task 1 - Raku](#task1)
- [PWC 211 - Task 2 - Raku](#task2)
- [PWC 211 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 211 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 211 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 211 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 211 - Task 1 - Raku Implementation

The first task was about finding out if a given matrix, i.e., an array of arrays, has the *topleft* to *bottomright* diagonal made by the same values. In raku the task is quite simple:

<br/>
<br/>
```raku
sub MAIN() {

    my @matrix = [ [4, 3, 2, 1],
                   [5, 4, 3, 2],
                   [6, 5, 4, 3],
                 ];

    my %diag;
    %diag{ @matrix[ $_ ][ $_ ] }++ for 0 ..^ @matrix.elems;
    'False'.say if ( %diag.keys.elems != 1 || %diag{ @matrix[ 0 ][ 0 ] } != @matrix.elems );
    'True'.say;

}

```
<br/>
<br/>

I use a `%diag` hash to store all the values I find moving from top left to bottom right, and I count how many values of the same type are there. Then, if the hash has more than one key, or the first topleft value (used as a key) has been counted less than the number of elements counted in total, it means that there are different values.


<a name="task2"></a>
## PWC 211 - Task 2 - Raku Implementation

I'm not sure I understood this task: splitting an array so that they both have the same average values.

<br/>
<br/>
```raku
sub MAIN( *@list where{ @list.elems == @list.grep( * ~~ Int ).elems } ) {

    for @list.permutations -> @current {
		# for 0 ..^ @current.elems {
		#     # find the first couple that gives the same average
		#     my ($left, $right) = @current[ 0 .. $_ ], @current[ $_ + 1 .. * ];
		#     if ( ( $left.sum / $left.elems ) == ( $right.sum / $right.elems ) ) {
		# 	say "{ $left.join( ',' ) } = { $left.sum / $left.elems }  and { $right.join( ',' ) } = { $right.sum / $right.elems } ";
		# 	exit;
		#     }
		# }

		my $split-at = ( @current.elems - 1 ) / 2;
		my ($left, $right) = @current[ 0 .. $split-at ], @current[ $split-at + 1  .. *  ];
		if ( ( $left.sum / $left.elems ) == ( $right.sum / $right.elems ) ) {
		    say "{ $left.join( ',' ) } = { $left.sum / $left.elems }  and { $right.join( ',' ) } = { $right.sum / $right.elems } ";
		    exit;
		}

    }
}

```
<br/>
<br/>

In the beginning, I came up with the commented out solution, that finds the first occurrence of a couple of arrays *of different sizes* that have the same average value.
However, according to the solution proposed as an example in the task, it seems to me that the two arrays must have the same size, so there is no need to dynamically split them since I always know I have to split the original array at its half.
I store the two arrays into `$left` and `$right` and see if their `sum`, divided by the number of elements, is the same.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 211 - Task 1 - PL/Perl Implementation

Same implementation as Raku: I use a `$diag` hash to count how many different values can be found on the diagonal, and then see if there is only one counted for the same number as the rows of the matrix.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc211.task1_plperl( int[][] )
RETURNS bool
AS $CODE$
   my ( $array ) = @_;

   my $diag;
   $diag->{ $array->[ $_ ]->[ $_ ] }++ for ( 0 .. scalar( $array->@* ) - 1 );
   return 0 if ( keys( $diag->%* ) != 1 );
   return 0 if ( $diag->{ $array->[ 0 ]->[ 0 ] } != scalar( $array->@* ) );
   return 1;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 211 - Task 2 - PL/Perl Implementation

Here I had to surrend and use `plperlu`: I needed a permutor, and so I decided to load the `List::Permutor` module. At that point, I also fired up `List::Util` to get an out of the box `sum` function.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc211.task2_plperl( int[] )
RETURNS SETOF int[]
AS $CODE$
   use List::Permutor;
   use List::Util qw/sum/;
   my ( $array ) = @_;

   my $permutator   = List::Permutor->new( $array->@* );
   while ( my @current = $permutator->next ) {
     my ( $split_at ) = ( $#current + 1) / 2;
     my ( $left, $right ) = ( [ @current[ 0 .. ( $split_at - 1 ) ] ], [ @current[ $split_at .. $#current ] ] );

     if ( ( sum( $left->@* ) / scalar( $left->@* ) ) == ( sum( $right->@* ) / scalar( $right->@* ) ) ) {
     	return_next( $left );
	    return_next( $right );
	    return undef;
     }
   }

return undef;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

For each `@current` permutation, I split the array into `$left` and `$right` and see if their `sum` divided by the number of elements is the same. If it is, I return both the arrays and terminate the function.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 211 - Task 1 - PL/PgSQL Implementation

I took a much more simpler, and less elegant, solution here: I decided to store the `previous_val` value found on a specific cell and compare it with the next one (i.e., next row, one column right). If the values do not match, there is no need to conitnue: the matrix is not good.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc211.task1_plpgsql( a int[][])
RETURNS bool
AS $CODE$
DECLARE
	current_row  int := 1;
	current_col  int := 1;
	previous_val int := NULL;
BEGIN
	WHILE current_row <= array_length( a, 1 ) LOOP
	      IF current_row > array_length( a, 2 ) THEN
	      	 RETURN false;
          END IF;

	      IF previous_val IS NULL THEN
	      	 previous_val := a[ current_row ][ current_row ];
	      ELSIF previous_val <> a[ current_row ][ current_row ] THEN
	      	   RETURN false;
          END IF;

	     current_row := current_row + 1;
	END LOOP;

	RETURN true;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Remember that in SQL arrays are numbered starting from 1.


<a name="task2plpgsql"></a>
## PWC 211 - Task 2 - PL/PgSQL Implementation

Here I used a  *permutation* function that can be found [online at the PostgreSQL Wiki](https://wiki.postgresql.org/wiki/Permutations){:target="_blank"}. I used such function as the `List::Permutor` module in the PL/Perl implementation, and every permutation was used in a cursor-like `FOR` loop.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc211.task2_plpgsql( a int[] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	split_at int := 0;
	current_array int[];
	l int[];
	r int [];
	avg_l numeric;
	avg_r numeric;
BEGIN
	split_at := array_length( a, 1 ) / 2;

	FOR current_array IN SELECT * FROM pwc211.permute( a ) LOOP
	    l := current_array[ 1:split_at ];
	    r := current_array[ (split_at + 1): array_length( a, 1 ) ];


	    SELECT avg( v )
	    INTO   avg_l
	    FROM   unnest( l ) v;

	    SELECT avg( v )
	    INTO   avg_r
	    FROM   unnest( r ) v;

	    IF avg_r = avg_l THEN
	       RETURN NEXT l;
	       RETURN NEXT r;
	       RETURN;
	    END IF;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I split by range operator (`:`) the array into two: `l` and `r`. The average is quite simple to compute, since the `avg()` function can be applied to the *tabify* array by means of `unnest`. If the average values are the same, the function can terminate returning both the arrays.
