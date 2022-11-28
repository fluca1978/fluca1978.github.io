---
layout: post
title:  "Perl Weekly Challenge 193: Map, map and remap!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 193: Map, map and remap!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 193](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0193/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)

<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)


Last, the solutions in PostgreSQL PL/PgSQL:

<br/>
- [Task 1 in PostgreSQL Pl/PgSQL](#task1plpgsql)
- [Task 2 in PostgreSQL Pl/PgSQL](#task2plpgsql)



<a name="task1"></a>
## PWC 193 - Task 1 - Raku Implementation

The first task was about finding out all binary numbers available with a specific number of digits, given as an input value.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {

    my @bins = ( 0 .. 2 ** $n - 1 ).map( { sprintf "%b", $_ } ).map( { '0' x ( $n - $_.chars ) ~ $_ } );

    @bins.join( ', ' ).say;
}
```
<br/>
<br/>

The available binary numbers, given the `$n` number of digits, is `2 ** $n`. I `map` the list of such integers numbers printing the result as `%b`, so that `sprintf` does the *integer to binary* translation for me. The result is then re-`map`ped appending zeros on the left for all the length of the number as required.



<a name="task2"></a>
## PWC 193 - Task 2 - Raku Implementation

Given a list of strings, print out all the strings that have only odd differences between a character and the one on its left. The translations are made considering that `a` is mapped to `0` and so on.

<br/>
<br/>
```raku
sub MAIN( *@s ) {
    my %translations = ( 'a' .. 'z' ).map( { state $counter = 0; $_ => $counter++ } );

    my %strings = @s.map( { $_ => $_.lc.comb.map( { %translations{ $_ }  } ) } );

    for %strings.kv -> $current-string, $current-array {
	my @difference;
	for 1 ..^ $current-array.elems {
	    @difference.push: $current-array[ $_ ] - $current-array[ $_ - 1 ];
	}

	$current-string.say if @difference.grep( { $_ !%% 2 } ).elems == @difference.elems;
    }

}

```
<br/>
<br/>

First of all, I create a `%translations` hash that is keyed by a letter and has the translating integer value.
Then I `map` the array of strings `@s` to another hash, named `%strings`, keyed by the input string and with an array of integer values for every letter.
I do loop over this last hash and then I do a nested loop over the array of integers computing the `@difference` of every couple of letters.
Last I `grep` the `@difference` array to count the number of elements that have an odd value, and if the length of the grepped array is the same as the initial array length it means that the string has only odd differences, so it is printed.


<a name="task1plperl"></a>
## PWC 193 - Task 1 - PL/Perl Implementation

A straightforward implementation in Perl of my Raku solution:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc193.task1_plperl( int )
RETURNS SETOF TEXT
AS $CODE$
  my ( $n ) = @_;

  my @bins =
       map { '0' x ( $n - length( $_ ) ) . $_ }
       map { sprintf "%b", $_ }
       ( 0 .. 2 ** $n - 1 );

  return_next( $_ ) for @bins;
  return;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 193 - Task 2 - PL/Perl Implementation

Again, a Perl implementation of the same Raku part proposed as solution:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc193.task2_plperl( text )
RETURNS text
AS $CODE$
 my ( $current_string ) = @_;
 my @chars = split '', $current_string;
 my $counter = 0;
 my %translations = map {  $_ => $counter++ } ( 'a' .. 'z' );

 my @values = map { $translations{ $_ } } @chars;

 my @difference;
 for my $index ( 1 .. length( $current_string )  ) {
   push @difference, $chars[ $_ ] - $chars[ $_ - 1 ];
 }

 if ( scalar( grep { $_ % 2 != 0 } @difference ) == @difference ) {
    return $current_string;
 }
 else {
   return undef;
 }

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


Please note that this function accepts a single string, not an array of strings, so that in order to complete the task you need to call (i.e., `SELECT`) the function joining the list of strings. But this is quite common in SQL.


<a name="task1plpgsql"></a>
## PWC 193 - Task 1 - PL/PgSQL Implementation

A simple implementation, assuming however to convert all numbers to `24` bit values:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc193.task1_plpgsql( n int )
RETURNS SETOF TEXT
AS $CODE$
DECLARE
	i int;
BEGIN
	FOR i IN 0 .. pow( 2, n ) - 1 LOOP
	    RETURN NEXT i::bit( 24 )::text;
	END LOOP;

	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 193 - Task 2 - PL/PgSQL Implementation

A more verbose implementation of the PL/Perl one.
<br/>
First I create a temporary table to store the translations of characters; I also create a temporary table to store the differences computed. Then I simply loops over every single char to extract the values, compute the differences and store them in a tuple into the table.
Last I compare the counting of all the computed differences with all the odd ones, and in the case they match, I return the string.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc193.task2_plpgsql( s text )
RETURNS text
AS $CODE$
DECLARE
	c char;
	pre int;
	cur int;

	count_all int;
	count_odd int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS translations( l char, i int DEFAULT 0 );
	TRUNCATE translations;

	INSERT INTO translations
	VALUES
	  ( 'a', 0 )
	, ( 'b', 1 )
	, ( 'c', 2 )
	, ( 'd', 3 )
	, ( 'e', 4 )
	, ( 'f', 5 )
	, ( 'g', 6 )
	, ( 'h', 7 )
	, ( 'i', 8 )
	, ( 'j', 9 )
	, ( 'k', 10 )
	, ( 'l', 11 )
	, ( 'm', 12 )
	, ( 'n', 13 )
	, ( 'o', 14 )
	, ( 'p', 15 )
	, ( 'q', 16 )
	, ( 'r', 17 )
	, ( 's', 18 )
	, ( 't', 19 )
	, ( 'u', 20 )
	, ( 'v', 21 )
	, ( 'x', 22 )
	, ( 'y', 23 )
	, ( 'z', 24 );


	CREATE TEMPORARY TABLE IF NOT EXISTS result( v int );
	TRUNCATE result;

	FOR c IN SELECT regexp_split_to_table( s, '' ) LOOP
	    SELECT i
	    INTO cur
	    FROM translations
	    WHERE l = c;

	    IF pre IS NOT NULL THEN
	       INSERT INTO result
	       SELECT cur - pre;
	    END IF;

	    pre := cur;
	END LOOP;


	SELECT count( * )
	INTO count_all
	FROM result;

	SELECT count(*)
	INTO count_odd
	FROM result
	WHERE V % 2 <> 0;

	IF count_all <> count_odd THEN
	   RETURN NULL;
	ELSE
	  RETURN s;
	END IF;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
