---
layout: post
title:  "Perl Weekly Challenge 204: Arrays everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 204: Arrays everywhere!

This post presents my solutions to the [Perl Weekly Challenge 204](https://perlweeklychallenge.org/blog/perl-weekly-challenge-204/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 204 - Task 1 - Raku](#task1)
- [PWC 204 - Task 2 - Raku](#task2)
- [PWC 204 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 204 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 204 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 204 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 204 - Task 1 - Raku Implementation

The first task required to find out if a given input array is *monotonic*: it increases or decreasing going from the first cell to the last one.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my $monotonic-type;
    for 0 ^..^ @list.elems {
	if ( ! $monotonic-type ) {
	    $monotonic-type = ( @list[ $_ ] > @list[ $_ - 1 ] ) ?? True !! False;
	}

	# elements are the same
	next if @list[ $_ ] == @list[ $_ - 1 ];

	'0'.say and exit if ( ! $monotonic-type && @list[ $_ ] > @list[ $_ - 1 ] );
	'0'.say and exit if (   $monotonic-type && @list[ $_ ] < @list[ $_ - 1 ] );
    }

    '1'.say;
}

```
<br/>
<br/>


Since I don't know in advance what type of monotomy I'm looking for, I use a flag `$monotonic-type` that remains unset until I find out the first elements that are not equal: then the flag is set to `True` or `False` depending on the direction.
Then, as soon as I find out a violation of the monotony, I exit the program.

<a name="task2"></a>
## PWC 204 - Task 2 - Raku Implementation

Reshape an input matrix.

<br/>
<br/>
```raku
ub MAIN( Int :$r, Int :$c, *@matrix ) {
    my @M = @matrix.map: {  $_.split( ' ' )  };

    # if cannot reshape, exit
    '0'.say and exit if ( $r * $c ) < ( @M.elems * @M[ 0 ].elems );

    my @N;
    my @new-row;
    for @M -> $row {

	for 0 ..^ $row.elems {
	    @new-row.push: $row[ $_ ] if ( @new-row.elems < $c );
	    @N.push: [ @new-row ] if ( @new-row.elems == $c );
	    @new-row = () if ( @new-row.elems == $c );

	}
    }

    @N.join( "\n" ).say;
}

```
<br/>
<br/>

The input `@matrix` is assumed to be a list of strings, where every value is separated by a space.
For instance, you can invoke the application with:

<br/>
<br/>
```shell
% raku raku/ch-2.p6 -r=3 -c=2 "1 2 3" "4 5 6"
```
<br/>
<br/>

I do create a new matrix `@N` and keep a `@new-row` that I populate as passing thru all the elements in the original matrix. Once the row has raised to the `$c` size, I push it into `@N` and start over.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 204 - Task 1 - PL/Perl Implementation

Very similar to the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc204.task1_plperl( int[] )
RETURNS int
AS $CODE$
 my ( $list ) = @_;
 my $monotonic_type;

 for ( 1 .. scalar( $list->@* ) - 1 ) {
   next if ( $list->[ $_ ] == $list->[ $_ - 1 ] );

   if ( ! defined( $monotonic_type ) ) {
      $monotonic_type = ( $list->[ $_ ] > $list->[ $_ - 1 ] ) ? 1 : 0;
   }

   return 0 if ( $monotonic_type && $list->[ $_ ] < $list->[ $_ - 1 ] );
   return 0 if ( ! $monotonic_type && $list->[ $_ ] > $list->[ $_ - 1 ] );
 }

 return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 204 - Task 2 - PL/Perl Implementation

Again, an implementation similar to the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc204.task2_plperl( int, int, int[][] )
RETURNS int[][]
AS $CODE$
  my ( $r, $c, $matrix ) = @_;
  my @N;
  my @new_row;

  return undef if ( ( $r * $c ) < $matrix->@* * $matrix->[0]->@* );

  for my $row ( 0 .. scalar( $matrix->@* ) - 1 ) {
     for my $col ( 0 .. scalar( $matrix->[ $row ]->@* ) - 1 ) {
        push @new_row, $matrix->[ $row ]->[ $col ] if ( @new_row < $c );
    	if ( @new_row == $c ) {
	      push @N, [ @new_row ];
	      @new_row = ();
	   }
     }
  }

  return [ @N ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

One nice thing is that PL/Perl does support nested arrays thru references automatically, so `$matrix` is an effective matrix.
Another thing to keep in mind is that PL/Perl requires you to return array references when dealing with complex structures.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 204 - Task 1 - PL/PgSQL Implementation

A straightforward implementation of the Raku one. Note how much more verbose it is this implementation!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc204.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
   monotonic_mode bool;
   i int;
BEGIN

	FOR i IN 2 .. array_length( l, 1 )  LOOP
	    IF l[ i ] = l[ i - 1 ] THEN
	       CONTINUE;
	    END IF;

	    IF monotonic_mode IS NULL THEN
	       IF l[ i ] > l[ i - 1 ] THEN
	       	  monotonic_mode := true;
	       ELSE
	         monotonic_mode := false;
	       END IF;
	    END IF;

           IF monotonic_mode AND l[ i ] < l[ i - 1 ] THEN
	      RETURN 0;
	   END IF;
           IF NOT monotonic_mode AND l[ i ] > l[ i - 1 ] THEN
	      RETURN 0;
	   END IF;
	END LOOP;

	RETURN 1;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 204 - Task 2 - PL/PgSQL Implementation

Similar to the previous implementations, but this time I do not return a matrix, but a `SET OF` arrays, so to simulate a matrix in a simple way.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc204.task2_plpgsql( r int, c int, a int[][] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	current_row int[];
	index_r int;
	index_c int;
BEGIN
	IF ( r * c ) < ( array_length( a, 1 ) * array_length( a, 2 ) ) THEN
	   RETURN;
	END IF;

	FOR index_r IN 1 .. array_length( a, 1 ) LOOP
	    FOR index_c IN 1 .. array_length( a, 2 ) LOOP
	    	current_row := current_row || a[ index_r ][ index_c ];
		IF array_length( current_row, 1 ) = c THEN
		   RETURN NEXT current_row;
		   current_row := array[]::int[];

		END IF;
	    END LOOP;
	END LOOP;

	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
