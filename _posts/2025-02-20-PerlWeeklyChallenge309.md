---
layout: post
title:  "Perl Weekly Challenge 309: Perl and Raku mainly"
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

# Perl Weekly Challenge 309: Perl and Raku mainly

This post presents my solutions to the [Perl Weekly Challenge 309](https://perlweeklychallenge.org/blog/perl-weekly-challenge-309/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 309 - Task 1 - Raku](#task1)
- [PWC 309 - Task 2 - Raku](#task2)
- [PWC 309 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 309 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 309 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 309 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 309 - Task 1 - Raku Implementation


Given a sorted array of integers, find out the integer that has the min differences from the previous one.


<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    my %gaps;
    for 1 ..^ @nums.elems {
		%gaps{ @nums[ $_ ] - @nums[ $_ - 1 ] } = @nums[ $_ ];
    }

    %gaps{ %gaps.keys.min }.say;
}

```
<br/>
<br/>


I populate an hash of `%gaps` with the difference between elements, and value as the element, so that it does just suffice to extract the min key of the has to get the value. Please note that duplicated differences are overwritten, so only the last smallest difference is kept.

<a name="task2"></a>
## PWC 309 - Task 2 - Raku Implementation

Given an array of integers, find out the smallest difference between any element of the array.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {

    my $min = Inf;
    for @nums -> $current {
		my $found = @nums
			     .grep( * != $current )
			     .map( { abs( $current - $_.Int ) } )
			     .min;
		$min = $found if $min > $found;
    }

    $min.say;
}

```
<br/>
<br/>


I iterate over element of the array, and then `grep` for all other elements, `map`ping each one to the difference and extracting the min value.
If the min is less than the initial one, or previous one, I keep this, otherwise go to the next element.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 309 - Task 1 - PL/Perl Implementation

Essentially, this is the same implementation as in Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc309.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   my $gaps = {};

   for my $index ( 1 .. $nums->@* - 1 ) {
       $gaps->{ $nums->@[ $index ] - $nums->@[ $index - 1 ] } = $nums->@[ $index ];
   }

   return $gaps->{ ( sort( keys $gaps->%* ) )[0] };

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 309 - Task 2 - PL/Perl Implementation

Same solution as in Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc309.task2_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   my $min;

   for my $current ( $nums->@* ) {

       my $current_min = (
       				sort
       				map { $_ > $current ? $_ - $current : $current - $_ }
       				grep { $_ != $current }
       				$nums->@* )[ 0 ];

      if ( ! $min || $current_min < $min ) {
      	 $min = $current_min;
      }
   }

   return $min;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


I use `sort` on the resulting list to get out the min value.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 309 - Task 1 - PL/PgSQL Implementation

A single query with window functions make the deal to find out, for each value, the difference with the next one, hence the min difference.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc309.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$

   WITH data AS (
   	SELECT v, v - lag( v, 1, v * -10 ) over () AS diff
	FROM unnest( nums ) v
	ORDER BY diff ASC
   )
   SELECT  v
   FROM data

   LIMIT 1;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 309 - Task 2 - PL/PgSQL Implementation

In this implementation I iterate over the array elements and compute the differences.



<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc309.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE

	diff int;
	current_diff int;
	l int;
	r int;
BEGIN
	FOR l IN SELECT v FROM unnest( nums ) v LOOP
	    for r IN SELECT v FROM unnest( nums ) v LOOP
	    	IF r = l THEN
		   CONTINUE;
		END IF;

	    	current_diff := r - l;
		IF current_diff < 0 THEN
		   current_diff := current_diff * -1;
		END IF;

		IF diff IS NULL OR current_diff < diff THEN
		   diff := current_diff;
		END IF;
	    END LOOP;

	END LOOP;

	RETURN diff;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
