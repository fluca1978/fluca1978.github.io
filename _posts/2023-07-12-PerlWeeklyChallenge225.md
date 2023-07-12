---
layout: post
title:  "Perl Weekly Challenge 225: I'm back"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 225: I'm back

This post presents my solutions to the [Perl Weekly Challenge 225](https://perlweeklychallenge.org/blog/perl-weekly-challenge-225/){:target="_blank"}.
<br/>
I know I've been not solving tasks for a while, a month or so, but the fact is that **I recently changed my main job** and therefore I was very busy in finishing up all the things at my previous job, as well as learning everything from scratch where I'm now!
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 225 - Task 1 - Raku](#task1)
- [PWC 225 - Task 2 - Raku](#task2)
- [PWC 225 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 225 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 225 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 225 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 225 - Task 1 - Raku Implementation

Given a list of sentences, where each word is separated by a single space, output the max counting of words in a sentence.

<br/>
<br/>
```raku
sub MAIN( *@sentences ) {
    my %data;
    %data{ $_ } = $_.split( /\s/ ).elems for @sentences;
    %data.values.max.say;
}

```
<br/>
<br/>

I soved with a simple hash that is keyed by the sentence and keep track of the word counting, then I select the max value.



<a name="task2"></a>
## PWC 225 - Task 2 - Raku Implementation

Given a list of integers, produce a *left-right* sum, which is kind of bizzarre.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems } ) {
    my ( @left, @right );

    @left.push: 0;
    @left.push: @numbers[ 0 .. $_ ].sum for 0 ..^ @numbers.elems - 1;

    @right.push: @numbers[ $_ .. * ].sum for 1 ..^ @numbers.elems;
    @right.push: 0;

    say @left;
    say @right;

    (@left Z[-] @right).map( { $_.abs } ).say;
}

```
<br/>
<br/>

I use the `Z[-]` operator to mix the arrays producing the difference, then I `map` the result into a only positive value.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 225 - Task 1 - PL/Perl Implementation

As usual, more verbose than its counterpart in Raku, but still a simple task. I keep track of the max value as long as I iterate across all the sentences:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc225.task1_plperl( text[] )
RETURNS int
AS $CODE$
   my ( $sentences ) = @_;

   my $max = 0;

   for ( $sentences->@* ) {
       my $count = scalar split( /\s/, $_ );
       $max = $max > $count ? $max : $count;
   }

   return $max;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The trick here is to count, by means of `scalar`, the words in a `split`ted sentence.


<a name="task2plperl"></a>
## PWC 225 - Task 2 - PL/Perl Implementation

A translation of the Raku implementation, but with a single final loop to zip the arrays:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc225.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $numbers ) = @_;

   my ( @left, @right );

   push @left, 0;
   for my $index ( 0 .. scalar( $numbers->@* ) - 2 ) {
       my $sum = 0;
       $sum += $_ for ( $numbers->@[ 0 .. $index ] );
       push @left, $sum;
   }

   for my $index ( 1 .. scalar( $numbers->@* ) - 1 ) {
       my $sum = 0;
       $sum += $_ for ( $numbers->@[ $index .. scalar( $numbers->@* )  ] );
       push @right, $sum;
   }

   for my $index ( 0 .. $#left ) {
       my $value = $left[ $index ] - $right[ $index ];
       $value = $value > 0 ? $value : $value * - 1;
       return_next( $value );
   }

return undef;


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Note that since I iterate on `@left` I don't need the final zero in the `@right` array.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 225 - Task 1 - PL/PgSQL Implementation

Similar to the PL/Perl implementation, but I use the SQL to count the words since I can split a text into a table:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc225.task1_plpgsql( sentences text[] )
RETURNS int
AS $CODE$
DECLARE
	current_sentence text;
	max_length int := 0;
	current_length int := 0;
BEGIN
	FOREACH current_sentence IN ARRAY sentences LOOP
		SELECT count(*)
		INTO current_length
		FROM regexp_split_to_table( current_sentence, ' ' );

		IF current_length > max_length THEN
		   max_length := current_length;
		END IF;
	END LOOP;

RETURN max_length;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 225 - Task 2 - PL/PgSQL Implementation

Similar to the PL/Perl implementation, makes usage of arrays and internal state.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc225.task2_plpgsql( numbers int[] )
RETURNS SETOF INT
AS $CODE$
DECLARE
	current_value int := 0;
	current_sum int := 0;

	current_index int := 0;
	current_sub_index int := 0;
	a_left int[];
	a_right int[];
BEGIN

	a_left := array_append( a_left, 0 );
	current_index := 0;
	FOR current_index IN 1 .. array_length( numbers, 1 ) - 1 LOOP
	    current_sum := 0;
	    FOR current_sub_index IN 1 .. current_index LOOP
	    	current_sum := current_sum + numbers[ current_sub_index ];
	    END LOOP;
	    a_left := array_append( a_left, current_sum );
	END LOOP;


	current_index := 0;
	FOR current_index IN 2 .. array_length( numbers, 1 )  LOOP
	    current_sum := 0;
	    FOR current_sub_index IN current_index .. array_length( numbers, 1 ) LOOP
	    	current_sum := current_sum + numbers[ current_sub_index ];
	    END LOOP;
	    a_right := array_append( a_right, current_sum );
	END LOOP;

        a_right := array_append( a_right, 0 );

	FOR current_index IN 1 .. array_length( a_left, 1 ) LOOP
	    current_value := a_left[ current_index ] - a_right[ current_index ];
	    IF current_value > 0 THEN
	       RETURN NEXT current_value;
	    ELSE
	       RETURN NEXT ( current_value * -1 );
	    END IF;
	END LOOP;

RETURN;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
