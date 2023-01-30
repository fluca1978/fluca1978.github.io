---
layout: post
title:  "Perl Weekly Challenge 202: nested loops everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 202: nested loops everywhere!

This post presents my solutions to the [Perl Weekly Challenge 202](https://perlweeklychallenge.org/blog/perl-weekly-challenge-202/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 202 - Task 1 - Raku](#task1)
- [PWC 202 - Task 2 - Raku](#task2)
- [PWC 202 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 202 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 202 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 202 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 202 - Task 1 - Raku Implementation

The first task was about searching for a natural sequence of odd numbers within an input array of integers.

<br/>
<br/>
```raku
sub MAIN( Bool :$verbose = False,
	*@list where { @list.grep( { $_ ~~ Int && $_ > 0 } ).elems == @list.elems } ) {
    my @odds;
    for @list {
		next if $_ %% 2;
		@odds.push: $_ and next if ( ! @odds );
		next if @odds.grep( $_ );
		next if $_ != ( @odds[ * - 1 ] + 2 );
		@odds.push: $_;
    }

    @odds.join( ', ' ).say if $verbose;
    '1'.say and exit if ( @odds.elems >= 3 );
    '0'.say;
}

```
<br/>
<br/>


The idea is quite simple: I iterate over the `@list` of numbers and add the first odd that I find into the `@odds` array. Then I seek for the next odd number, that is the last one I found added by `2`. When I find the next odd number, I add it to the `@odds` array.
Last I test if the `@odds` array has a length greater or equal to `3`.


<a name="task2"></a>
## PWC 202 - Task 2 - Raku Implementation

This task is a little more complex to understand: find out the largest left *valley* within a number array.
A valley is a sequence made by two parts, left and right, where the left is not increasing and the right is not decreasing.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( { $_ > 0 && $_ ~~ Int } ).elems == @list.elems } ) {

    my %valleys;

    for 0 ..^ @list.elems - 1 -> $index {
	my $current = @list[ $index ];
	next if @list[ $index + 1 ] > $current;  # increasing!

	my @valley-left;
	@valley-left.push: $current;
	for $index ^..^ @list.elems {
	    my $previous = @valley-left[ * - 1 ];
	    @valley-left.push: @list[ $_ ] if ( @list[ $_ ] <= $previous );
	    last if @list[ $_ ] > $previous;
	}

	my @valley-right;

	if ( $index + @valley-left.elems  < @list.elems ) {
	    @valley-right.push: @list[ $index + @valley-left.elems ];
	    for $index + @valley-left.elems ^..^ @list.elems {
		my $previous = @valley-right[ * - 1 ];
		@valley-right.push: @list[ $_ ] if ( @list[ $_ ] >= $previous );
		last if @list[ $_ ] < $previous;
	    }
	}

	%valleys{ @valley-left.elems + @valley-right.elems } = [ |@valley-left, |@valley-right ];
    }

    %valleys{ %valleys.keys.max }.join( ', ' ).say;

}

```
<br/>
<br/>

My implementation is really verbose.
The idea is that I seek for the beginning of the *partitioning* looking for the first two numbers where the latter is greater than the former. This clearly begins a left side partitioning.
Then I loop over the list to extract the widest array where there is not an increasing set of values, and I place every value into the `@valley-left` array. Then I do continue from there and seek for the right partition, placing all values into the `@valley-right`.
The result, that is the concatenation between `@valley-left` and `@valley-right`, is placed into an hash named `%valleys` keyed by the size of the resulting concatenation.
Last, I extract the value having the highest key.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 202 - Task 1 - PL/Perl Implementation

A mere implementation of the Raku approach:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc202.task1_plperl( int[] )
RETURNS int
AS $CODE$
  my ( $list ) = @_;
  my @odds;

  for ( $list->@* ) {
    next if $_ % 2 == 0;

    push( @odds, $_ ) and next if ! @odds;
    next if $_ != ( $odds[ -1 ] + 2 );
    push( @odds, $_ );
  }

  return 1 if @odds >= 3;
  return 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 202 - Task 2 - PL/Perl Implementation

Same implementation as the Raku one, with the only difference that I keep track of the `$largest` concatenation found, so that I can lookup quickly.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc202.task2_plperl( int[] )
RETURNS int[]
AS $CODE$
  my ( $list ) = @_;
  my ( %valleys );
  my $largest = 0;

  for my $index ( 0 .. scalar( $list->@* ) - 1 ) {
     my $current = $list->[ $index ];
     next if $list->[ $index + 1 ] > $current;

     my ( @valley_left ) = ( $current );
     for ( $index + 1 .. scalar( $list->@* ) - 1 ) {
         my $previous = $valley_left[ -1 ];
     	 push( @valley_left, $list->[ $_ ] ) if ( $list->[ $_ ] <= $previous );
	 last if $list->[ $_ ] > $previous;
     }


     my @valley_right;
     if ( $index + scalar( @valley_left ) < scalar( $list->@* ) ) {
     	my $previous = $list->[ $index + scalar( @valley_left ) ];
	@valley_right = ( $previous );
	for ( $index + scalar( @valley_left ) + 1 .. scalar( $list->@* ) - 1 ) {
	    my $previous = $valley_right[ -1 ];
     	    push( @valley_right, $list->[ $_ ] ) if ( $list->[ $_ ] >= $previous );
	    last if $list->[ $_ ] < $previous;
	}
     }



     $valleys{ scalar( @valley_right ) + scalar( @valley_left ) } = [ @valley_left, @valley_right ];
     $largest = scalar( @valley_right ) + scalar( @valley_left ) if ( scalar( @valley_right ) + scalar( @valley_left ) > $largest );

  }

  return $valleys{ $largest };
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 202 - Task 1 - PL/PgSQL Implementation

Same implementation as in the previous tasks, with the only difference to keep in mind that in SQL arrays starts at index `1`!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc202.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
	odds int[];
	cur int;
BEGIN
	FOREACH cur IN ARRAY l LOOP
		IF cur % 2 = 0 THEN
		   CONTINUE;
		END IF;

		IF array_length( odds, 1 ) = 0 OR odds IS NULL THEN
		   odds := odds || cur;
		   CONTINUE;
		END IF;

		IF odds[ array_length( odds, 1 ) ] + 2 <> cur THEN
		   CONTINUE;
		END IF;

   	        odds := odds || cur;
	END LOOP;

	IF array_length( odds, 1 ) >= 3 THEN
	   RETURN 1;
	ELSE
	  RETURN 0;
	END IF;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 202 - Task 2 - PL/PgSQL Implementation

Same implementation of Raku, with the usage of a table instead of an hash.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc202.task2_plpgsql( l int[] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	cur int;
	lft int[];
	rgt int[];
	idx int;
	iter int;
	prev int;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS pwc202
	( lft int[], rgt int[], dim int DEFAULT 0 );
	TRUNCATE pwc202;


	FOR idx IN 1 .. array_length( l, 1 ) - 1 LOOP
		cur := l[ idx ];
		IF l[ idx + 1 ] > cur THEN
		   CONTINUE;
		END IF;

		lft := NULL;
		lft := array_append( lft, cur );
		FOR iter IN idx + 1 .. array_length( l, 1 ) - 1 LOOP
		    prev := lft[ array_length( lft, 1 ) ];
		    IF l[ iter ] <= prev THEN
		       lft := array_append( lft, l[ iter ] );
		    END IF;
		    EXIT WHEN l[ iter ] > prev;
		END LOOP;

		rgt := NULL;
		IF array_length( lft, 1 ) + idx <= array_length( l, 1 ) THEN
		   prev := l[ idx + array_length( lft, 1 ) ];
		   rgt := array_append( rgt, prev );
		   FOR iter IN array_length( lft, 1 ) + idx + 1 .. array_length( l, 1 )  LOOP
		      prev := rgt[ array_length( rgt, 1 ) ];
		      IF l[ iter ] >= prev THEN
		       rgt := array_append( rgt, l[ iter ] );
		      END IF;
		      EXIT WHEN l[ iter ] < prev;
		   END LOOP;
		END IF;

		INSERT INTO pwc202
		VALUES( lft, rgt, array_length( lft, 1 ) + array_length( rgt, 1 ) );



	END LOOP;

       RETURN QUERY SELECT array_cat( p.lft, p.rgt )
       FROM pwc202 p
       ORDER BY dim DESC
       LIMIT 1;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The temporary table `pwc202` is used as an hash to store the left and right partitions, and last I return the result of a query that extracts only one record with the highest size.
