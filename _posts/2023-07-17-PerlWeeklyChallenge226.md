---
layout: post
title:  "Perl Weekly Challenge 226: Array Indexes everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 226: Array Indexes everywhere!

This post presents my solutions to the [Perl Weekly Challenge 226](https://perlweeklychallenge.org/blog/perl-weekly-challenge-226/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 226 - Task 1 - Raku](#task1)
- [PWC 226 - Task 2 - Raku](#task2)
- [PWC 226 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 226 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 226 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 226 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 226 - Task 1 - Raku Implementation

The first task was about rearranging a given word by a set of indexes, where each index represents the new position of the original character in the word.


<br/>
<br/>
```raku
sub MAIN( Str $string, *@indexes
			 where { @indexes.grep( * ~~ Int ).elems == @indexes.elems && @indexes.elems == $string.chars }
	) {
    my $index = 0;
    my %letters;
    %letters{ @indexes[ $index++ ] } = $_  for $string.comb;

    %letters{ @indexes.sort }.join.say;

}

```
<br/>
<br/>

The main idea is to map each letter to its index, the given index out of the array, and the to rearrange the `%letters` values on the sorted array.

<a name="task2"></a>
## PWC 226 - Task 2 - Raku Implementation

Given an array of integers, find out how many steps are required to fill it with zeros assuming at each step you can only decrease any non-zero value of the minimum value contained in the array.

<br/>
<br/>
```raku
sub MAIN( *@numbers is copy where { @numbers.grep( * ~~ Int && * >= 0 ).elems == @numbers.elems } ) {
    my $moves;

    while ( @numbers.grep( * == 0 ).elems != @numbers.elems ) {
		my $removing = @numbers.grep( * > 0 ).min;
		$moves++;
		for 0 ..^ @numbers.elems {
		    next if ! @numbers[ $_ ];
		    @numbers[ $_ ] -= $removing;
		}
    }

    $moves.say;
}

```
<br/>
<br/>

The implementation loops while the `@numbers` array has at least a non zero element. Then, at each iteration, the system computes the minimum value to `$removing` from the other elements, and then loops on the `@numbers` array to decrease any non zero value of the given `$removing` value.
Every time a non zero `$removing` value is found, the coutning of the `$moves` is increased.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 226 - Task 1 - PL/Perl Implementation

Very similar to the Raku approach.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc226.task1_plperl( text, int[] )
RETURNS text
AS $CODE$
   my ( $string, $indexes ) = @_;
   my ( $index ) = 0;
   my $letters = {};

   for  ( split( //, $string ) ) {
       $letters->{ $indexes->[ $index++ ] } = $_;
   }

   my @chars;
   push @chars, $letters->{ $_ }  for ( sort( $indexes->@* ) );
   return join( '', @chars );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I use an anonymous hash `$letters` to keep track of each letter and its wanted index, then I rebuild an array of `@chars` with the sorted indexes and their corresponding letters. The result is joined and returned to the caller.


<a name="task2plperl"></a>
## PWC 226 - Task 2 - PL/Perl Implementation

A much more verbose implementation than the Raku one, but the main idea remains the same.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc226.task2_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $numbers ) = @_;
   my $moves = 0;

   # inner function to get the min value
   # non zero in the array
   my $min = sub {
      my $min = undef;
      for ( $_[0]->@* ) {
      	  next if $_ == 0;
      	  $min = $_ if ! $min || $_ < $min;
      }

      return $min;
   };

   # inner function to see if the array
   # if full of zeros
   my $is_empty = sub {
      my ( $array ) = @_;
      return scalar( grep( { $_ == 0 } $array->@* ) ) == scalar( $array->@* );
   };




   while ( ! $is_empty->( $numbers ) ) {
	 my $removing = $min->( $numbers );
	 $moves++;

	 for my $index ( 0 .. $numbers->@* ) {
	     next if $numbers->[ $index ] == 0;
	     $numbers->[ $index ] -= $removing;
	 }
   }


   return $moves;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

First of all, the function defines two anonymous utility functions to compute the non-zero minimum in the array, and to understand if the array is filled with all zeros.
<br/>
The main loop checks if the array is empty, and if it is not, gets the minimum value and decreases any non-zero elemnt in the array.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 226 - Task 1 - PL/PgSQL Implementation

Similar to the Raku implementation, but using a temporary table to handle the letters and their position.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc226.task1_plpgsql( word text, indexes int[] )
RETURNS text
AS $CODE$
DECLARE
	i int := 1;
	final_word text := '';
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS  word( letter char, original_index int );
	TRUNCATE word;

	INSERT INTO word( letter, original_index )
	SELECT l, row_number() over ()
	FROM regexp_split_to_table( word, '' ) l;

	FOREACH i IN ARRAY indexes LOOP
		SELECT final_word || l.letter
		INTO final_word
		FROM word l
		WHERE l.original_index = i;
	END LOOP;

	RETURN final_word;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

In the table `word` I store each letter with its position. the natural position. Then I loop over the given array and extract the letter at the given index, so to concatenate the final string.


<a name="task2plpgsql"></a>
## PWC 226 - Task 2 - PL/PgSQL Implementation

Similar to the previous implementations.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc226.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	moves int := 0;
	removing int := 0;
	i int;
BEGIN

	FOUND := true;

	WHILE FOUND LOOP
		-- get the nex min value
		SELECT min( n )
		INTO removing
		FROM unnest( nums ) n
		WHERE n > 0;

		-- stop (?)
		IF NOT FOUND OR removing IS NULL THEN
		   EXIT;
		ELSE
	           moves := moves + 1;
		END IF;

		FOR i IN 1 .. array_length( nums, 1 ) LOOP
		    IF nums[ i ] = 0 THEN
		       CONTINUE;
		    END IF;

		    nums[ i ] = nums[ i ] - removing;
		END LOOP;
	END LOOP;

	RETURN moves;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I select the minimum element from the array using an `unnest`ed array as a table; once the `removing` element is not found (i.e., is `NULL`) I end the main `WHILE` loop. Otherwise, I iterate over the array and decrement any non-zero element, after having incremented the `moves` variable by one unit.
