---
layout: post
title:  "Perl Weekly Challenge 233: Sorting, by similarity and frequency"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 233: Sorting, by similarity and frequency

This post presents my solutions to the [Perl Weekly Challenge 233](https://perlweeklychallenge.org/blog/perl-weekly-challenge-233/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 233 - Task 1 - Raku](#task1)
- [PWC 233 - Task 2 - Raku](#task2)
- [PWC 233 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 233 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 233 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 233 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 233 - Task 1 - Raku Implementation

The first task was about finding *similar words*, words made by the same amount and value of letters. This is quite simple, after all: extract all the letters from a word, sort them, so to create a kind of *unique* key, and append each word within the same key.

<br/>
<br/>
```raku
sub MAIN( *@words where { @words.elems > 1  } ) {
    my %similars;
    for @words -> $word {
		my $key = $word.comb.sort.join;
		%similars{ $key }.push: $word;
    }

    say 'Similar words: ' ~ $_.join( ' <-> ' ) for %similars.values.grep( *.elems > 1 );
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 233 - Task 2 - Raku Implementation

The second task was about sorting numbers by means of their decreasing frequency of appearence in the input.
There is the need for a kind of bag to this aim.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ) } ) {
    my %sorting;
    for @nums -> $current {
		my $frequency = @nums.grep( * == $current ).elems;
		%sorting{ $frequency } //= Array.new;
		%sorting{ $frequency }.push: $current if ( ! %sorting{ $frequency }.grep( * == $current ) );
    }

    %sorting{ $_ }.sort.join( ',' ).say for %sorting.keys.sort;

}

```
<br/>
<br/>

Every cell of `%sorting` is keeping an array of numbers, and is keyed by the `$frequency`, that is obtained by `grep`ping and counting all the elements in the input array.
In the end, printing out the values corresponding to the sorted keys (i.e., sorted frequencies) does the trick.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 233 - Task 1 - PL/Perl Implementation

The implementation follows the same idea of the Raku one: use an hash `$similars` that handles the sorted list of letters as the key, and the list of words as values. Then cross the hash with the key and, in the case the list is made by more than one word, return all of them.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc233.task1_plperl( text[] )
RETURNS SETOF text
AS $CODE$
   my ( $words ) = @_;

   my $similars = {};

   for my $word ( $words->@* ) {
       my $key = join '', sort split //, $word;
       push $similars->{ $key }->@*, $word;
   }

   for my $key ( keys $similars->%* ) {
       next if $similars->{ $key }->@* <= 1;
       return_next( $_ ) for ( $similars->{ $key }->@* );
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 233 - Task 2 - PL/Perl Implementation

Same implementation as the Raku solution, only a little more verbose.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc233.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   my $freqs = {};

   for my $current ( $nums->@* ) {
       my $f = grep { $_ == $current } $nums->@*;
       push $freqs->{ $f }->@*, $current;
   }

   for my $f ( sort keys $freqs->%* ) {
       return_next( $_ ) for ( sort $freqs->{ $f }->@* );
   }
   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 233 - Task 1 - PL/PgSQL Implementation

Implemented with a couple of queries and a temporary table.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc233.task1_plpgsql( words text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	current_word text;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS sorting( key text, word text );
	TRUNCATE sorting;

	FOREACH current_word IN ARRAY words LOOP
		INSERT INTO sorting( key, word )
		SELECT string_agg( k::text, '' )::text, current_word
		FROM ( SELECT v::text
		       FROM regexp_split_to_table( current_word, '' ) v
		       ORDER BY 1
		     ) k
	       ;
	END LOOP;

	RETURN QUERY
	       SELECT word
	       FROM   sorting
	       WHERE  key IN ( SELECT key
	       	      	       FROM sorting
			       GROUP BY key
			       HAVING COUNT(*) > 1 );

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


In the beginning there is the creation of the temporary table and its truncation, to prevent dirty data from previous executions.
Then a single query inserts into the table the ordered letters of the word, and the word itself.
Last, a query returns the words out of the table, having selected only those keys that appears more than once.

<a name="task2plpgsql"></a>
## PWC 233 - Task 2 - PL/PgSQL Implementation

Room for an *UPSERT* here! Again, use a temporary table as storage.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc233.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	current_number int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS freqs( v int, f int DEFAULT 1, primary key ( v ) );
	TRUNCATE freqs;

	FOREACH current_number IN ARRAY nums LOOP
		INSERT INTO freqs( v )
		VALUES( current_number )
		ON CONFLICT ( v )
		DO UPDATE
		   SET f = ( SELECT f + 1
		       	     FROM freqs
			     WHERE v = EXCLUDED.v )
		;
	END LOOP;

	RETURN QUERY
	       SELECT v
	       FROM freqs
	       ORDER BY f DESC, v DESC;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


I traverse the array of numbers, and insert into the table every number. In the case the number is already present, I use the *UPSERT* capability to update the frequency field instead. Therefore, the result, is to return the records out of the table having sorted them by frequency.
