---
layout: post
title:  "Perl Weekly Challenge 229: delete if unsorted, keep if multiple occurrencies"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 229: delete if unsorted, keep if multiple occurrencies

This post presents my solutions to the [Perl Weekly Challenge 229](https://perlweeklychallenge.org/blog/perl-weekly-challenge-229/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 229 - Task 1 - Raku](#task1)
- [PWC 229 - Task 2 - Raku](#task2)
- [PWC 229 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 229 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 229 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 229 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 229 - Task 1 - Raku Implementation

The first task was about computing how many words to delete from a given list, given that a word has to be deleted if its chars are not in order.

<br/>
<br/>
```raku
sub MAIN( *@words is copy ) {
    my $deleted = 0;

    for 0 ..^ @words.elems {
		my $word = @words[ $_ ];
		@words[ $_ ]:delete and $deleted++ if $word ne $word.comb.sort.join;
    }

    $deleted.say;
}

```
<br/>
<br/>

The idea is simple: I split the word into letters, sort and rejoin. If the result is different from the original word, such word has to be deleted.


<a name="task2"></a>
## PWC 229 - Task 2 - Raku Implementation

The second task was about finding out elements replicated in at least two out of three arrays.

<br/>
<br/>
```raku
sub MAIN() {
    my @array1 = 1, 1, 2, 4;
    my @array2 = 2, 4;
    my @array3 = 4;

    my %bag;
    %bag{ $_ }++ for @array1.unique;
    %bag{ $_ }++ for @array2.unique;
    %bag{ $_ }++ for @array3.unique;

    %bag.keys.grep( { %bag{ $_ } >= 2 } ).join( ',' ).say;
}

```
<br/>
<br/>


I use `%bag` to enumerate the values, taking care of `unique` values only.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 229 - Task 1 - PL/Perl Implementation

Same implementation as Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc229.task1_plperl( text[])
RETURNS int
AS $CODE$
   my ( $words ) = @_;
   my $deleted = 0;

   for my $word ( $words->@* ) {
       my $sorted = join( '', sort( split( //, $word ) ) );
       $deleted++ if ( $word ne $sorted );
   }

   return $deleted;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 229 - Task 2 - PL/Perl Implementation

Same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc229.task2_plperl( int[], int[], int[] )
RETURNS SETOF int
AS $CODE$
   my $bag = {};

   for my $array ( @_ ) {
       my $uniq = {};
       for my $item ( $array->@* ) {
       	   $uniq->{ $item }++;
       	   $bag->{ $item }++ if ( $uniq->{ $item } == 1 );
       }
   }

   for my $item ( keys $bag->%* ) {
       return_next( $item ) if ( $bag->{ $item } >= 2 );
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


Here I iterate over each incoming array, and use an utility hash `$uniq` to keep only unique values for the current array, placing them into the `$bag` hash.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 229 - Task 1 - PL/PgSQL Implementation

Similar to the other implementations, but this time I use a query to obtain the sorted word.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc229.task1_plpgsql( words text[] )
RETURNS int
AS $CODE$
DECLARE
	current_word text;
	sorted_word text;
	deleted int := 0;
BEGIN

	FOREACH current_word IN ARRAY words LOOP
		WITH letters( l ) AS (
		     SELECT regexp_split_to_table( current_word, '' )::text
		     ORDER BY 1
		)
		SELECT string_agg( l, '' )
		INTO sorted_word
		FROM letters;

		IF sorted_word <> current_word THEN
		   deleted := deleted + 1;
		END IF;
	END LOOP;

	RETURN deleted;
END

$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The `regexp_split_to_table` splits the word into a table, that is then reaggregated with `string_agg`.


<a name="task2plpgsql"></a>
## PWC 229 - Task 2 - PL/PgSQL Implementation

Use a temporary table to store every element of the array, grouping them and counting those with multiple occurrencies.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc229.task2_plpgsql( aa int[], ab int[], ac int[] )
RETURNS SETOF INT
AS $CODE$
DECLARE
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS bag( v int, c int, a text );
	TRUNCATE TABLE bag;

	INSERT INTO bag( v, c, a )
	SELECT v, count(*), 'a'
	FROM unnest( aa ) v
	GROUP BY 1,3;

	INSERT INTO bag( v, c, a )
	SELECT v, count(*), 'b'
	FROM unnest( ab ) v
	GROUP BY 1,3;

	INSERT INTO bag( v, c, a )
	SELECT v, count(*), 'c'
	FROM unnest( ac ) v
	GROUP BY 1,3;

	RETURN QUERY
	SELECT v
	FROM bag
	WHERE c = 1
	GROUP by v
	HAVING sum( c ) >= 2;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
