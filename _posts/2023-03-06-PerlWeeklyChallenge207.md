---
layout: post
title:  "Perl Weekly Challenge 207: arrays everuwhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 207: arrays everuwhere!

This post presents my solutions to the [Perl Weekly Challenge 207](https://perlweeklychallenge.org/blog/perl-weekly-challenge-207/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 207 - Task 1 - Raku](#task1)
- [PWC 207 - Task 2 - Raku](#task2)
- [PWC 207 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 207 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 207 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 207 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 207 - Task 1 - Raku Implementation

Given the keyboard layout, find out which words can be written using a single line of keys.
This is something `Vi` users should love (I mean, the home row!).

<br/>
<br/>
```raku
sub MAIN( *@words ) {
    my @keyboard = qw/ qwertyuiop asdfghjkl zxcvbnm /;

    for @words -> $current-word {
	  for @keyboard -> $current-row {
	    my $found = 0;
	    for $current-word.lc.comb -> $current-letter {
		    $found++ if ( $current-row ~~ /$current-letter/  );
	    }

	    $current-word.say if $current-word.chars == $found;
	  }
    }

}
```
<br/>
<br/>

The idea is to count how many letters do match a single row of the keyboard for a given input word. If the number of letters is the same as that counting the number of keys into the same row, the word can be written with only keys in a single row, and therefore it is printed.


<a name="task2"></a>
## PWC 207 - Task 2 - Raku Implementation

The first task is about computing an *H-index*, that is something I'm supposed to know quite well, being this the index of citations of scinetific papers. The idea is that the *H-index* is the number of articles that cites your own article.
<br/>
The script accepts a list of citations, and I need to compute how many articles have the common minimum number of citations.



<br/>
<br/>
```raku
sub MAIN( *@citations where { @citations.grep( * ~~ Int ).elems == @citations.elems } ) {
    @citations.sort.reverse.pairs.first( { $_.key >= $_.value  } ).key.say;
}

```
<br/>
<br/>

The solution requires a little explaination: I `sort` in a `reverse` order the array of citations, so that I start the array from the highest value. Then I get the `pairs` (i.e., the keys and the values) and extract the `first` value where the key is greater than the value, and I print it.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 207 - Task 1 - PL/Perl Implementation

Same as the Raku implementation. The solution is more verbose due to the lack of some operators.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc207.task1_plperl( text[] )
RETURNS SETOF text
AS $CODE$

   my ( $words ) = @_;
   my ( @keyboard ) = qw/ qwertyuiop asdfghjkl zxcvbnm /;

   for my $current_word ( $words->@* ) {
       for my $current_row ( @keyboard ) {
       	   my $found = 0;
	   	   for my $current_letter ( split( '', lc( $current_word ) ) ) {
	   	       $found++ if ( $current_row =~ /$current_letter/ );
	   	   }

	   	   if ( scalar( split( '', $current_word ) ) == $found ) {
	   	      return_next( $current_word );
	   	      last;
	   	   }
       }
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 207 - Task 2 - PL/Perl Implementation

Similar to the Raku approach: I get the keys and the values by means of `each`, and push the key into the `@data` array only if the key is greater than the value. Last, I do print the first element in the `@data` array.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc207.task2_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $citations ) = @_;
   my @cits = reverse sort $citations->@*;
   my @data;
   while ( my ( $key, $value ) = each( @cits ) ) {
   	 push @data, $key if ( $key >= $value );
   }

   return ( sort( @data ) )[ 0 ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 207 - Task 1 - PL/PgSQL Implementation

A much more verbose implementation.
I use a temporary table to store the keyboard layouts, then I iterate over all the words in input.
For each word, I do a join between the letters of the word and the letters within the keyboard input line. If the count of join is equal to the length of the word, the word can be written with such keyboard line.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc207.task1_plpgsql( w text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	current_word text;
	current_row  text;
	letters_found int;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS keyboard( k text );
	TRUNCATE keyboard;
	INSERT INTO keyboard( k )
	VALUES( 'qwertyuiop' ), ( 'asdfghjkl' ), ( 'zxcvbnm' );

	FOREACH current_word IN ARRAY w LOOP
		FOR current_row IN SELECT k FROM keyboard LOOP
			letters_found := 0;

			SELECT count(*)
			INTO   letters_found
			FROM   regexp_split_to_table( current_word, '' ) ww
			JOIN   regexp_split_to_table( current_row, '' ) kk
			ON     ww = kk;

			IF letters_found = length( current_word ) THEN
			   RETURN NEXT current_word;
			   EXIT;
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



<a name="task2plpgsql"></a>
## PWC 207 - Task 2 - PL/PgSQL Implementation

A single window function query:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc207.task2_plpgsql( citations int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
BEGIN
	RETURN QUERY WITH d AS (
	   SELECT c, row_number() OVER ( ORDER BY c desc ) r
	   FROM   unnest( citations ) c
	)
	SELECT MIN( c )
	FROM   d
	WHERE  r >= c
	;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The idea is to materialize the citations with their sorting key `r`, when find the first minimum value with the valeu less than the key.
