---
layout: post
title:  "Perl Weekly Challenge 205: nested loops in a rush!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 205: nested loops in a rush!

This post presents my solutions to the [Perl Weekly Challenge 205](https://perlweeklychallenge.org/blog/perl-weekly-challenge-205/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 205 - Task 1 - Raku](#task1)
- [PWC 205 - Task 2 - Raku](#task2)
- [PWC 205 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 205 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 205 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 205 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 205 - Task 1 - Raku Implementation

The first task was about finding out the *third max value* within an array, if any, or *the highest value*. Simple enough, but selecting which value to extract from the sorted array required me to pass thru an intermediate list.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {

    my @highests = @list.unique.sort;
    @highests[ * - ( @highests.elems > 2 ?? 3 !! 1 ) ].say;
}

```
<br/>
<br/>

The idea is to have `@list` sorted and filtered so that identical values are accounted only once. Then if the resulting `@highests` array has more than three elements, I can grab the *third* maximum value, otherwise I grab the absolute max.


<a name="task2"></a>
## PWC 205 - Task 2 - Raku Implementation

Finding the highest `xor` within an array. I used a double nested loop to find out the list of `xor`-ed values, and then grab the maximum value.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my @xors;
    for 0 ..^ @list.elems -> $left {
	  for $left ^..^ @list.elems -> $right {
	    @xors.push: @list[ $left ] +^ @list[ $right ];
	  }
    }

    @xors.max.say;
}

```
<br/>
<br/>

I spent a lot of time finding out why the `xor` operator (either `xor` or `^^`) was not working, to find out that **`+^`** was the right operator to use for *integer xor*.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 205 - Task 1 - PL/Perl Implementation

A little more verbose implementation than the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc205.task1_plperl( int[] )
RETURNS int
AS $CODE$
  my ( $list ) = @_;
  my $values = {};
  $values->{ $_ }++ for ( sort $list->@* );
  my $offset = keys( $values->%* ) > 2 ? -3 : -1;
  return ( sort( keys( $values->%* ) ) )[ $offset ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Since I don't have a *unique* function at hand, I place the values into an hash using the values as keys, so that I eliminate duplicates.
Then I compute an `$offset` to find out the max value to grab, and return it.



<a name="task2plperl"></a>
## PWC 205 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, but I simply store the max value into a scalar `$max` value.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc205.task2_plperl( int[] )
RETURNS int
AS $CODE$
  my ( $list ) = @_;
  my $max = 0;

  for my $left ( 0 .. $list->@* ) {
      for my $right ( $left + 1 .. $list->@* ) {
      	  my $xor = $list->[ $left ] ^ $list->[ $right ];
   	      $max = $xor if ( $xor > $max );
      }
  }

  return $max;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 205 - Task 1 - PL/PgSQL Implementation

Toward an SQL approach, I used a temporary table as a list of sorted elements. Then I count the number of tuples and decide how to limit the result set.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc205.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
	has_third_max int := 0;
	third_max     int := 0;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS t_sort( t int );
	TRUNCATE t_sort;

	INSERT INTO t_sort
	SELECT t
	FROM unnest( l ) t
	GROUP BY t
	HAVING count(*) = 1;

	SELECT count(*)
	INTO has_third_max
	FROM t_sort;

	IF has_third_max > 3 THEN
	   SELECT t
	   INTO   third_max
	   FROM t_sort
	   ORDER BY t DESC
	   OFFSET 3
	   LIMIT 1;
	ELSE
	   SELECT t
	   INTO   third_max
	   FROM t_sort
	   ORDER BY t DESC
	   LIMIT 1;
       END IF;

       RETURN third_max;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 205 - Task 2 - PL/PgSQL Implementation

Since doing the `xor` in SQL is not as trivial as it could seem, I used a trick and invoked the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc205.task2_plpgsql( l int[] )
RETURNS int
AS $CODE$
   SELECT pwc205.task2_plperl( l );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
