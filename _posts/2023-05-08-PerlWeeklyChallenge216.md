---
layout: post
title:  "Perl Weekly Challenge 216: words, grep and letters!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 216: words, grep and letters!

This post presents my solutions to the [Perl Weekly Challenge 216](https://perlweeklychallenge.org/blog/perl-weekly-challenge-216/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 216 - Task 1 - Raku](#task1)
- [PWC 216 - Task 2 - Raku](#task2)
- [PWC 216 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 216 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 216 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 216 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 216 - Task 1 - Raku Implementation

The first task was to select, from an input list of words, only those that have letters all included into a last specified *code word*.

<br/>
<br/>
```raku
sub MAIN( *@strings is copy ) {
    my @registration-code = @strings.pop.comb;

    # first implementation
    for @strings -> $word {
		my @result.push: @registration-code.grep( $_ ) for $word.comb;
		say $word if @result.join ~~ $word;
    }

    # second implementation
    my $sorted-registration-code = @registration-code.sort.join;
    for @strings -> $word {
		say $word if ( $sorted-registration-code ~~ / ^  { $word.comb.sort.join } / );
    }

}

```
<br/>
<br/>

I propose two different implementations: the first one uses a nested `for` loop to `grep` every letter into a word. The second implementation uses a regular expression and the sorting of the letters in the words to match.


<a name="task2"></a>
## PWC 216 - Task 2 - Raku Implementation

The second task was about finding out which *stickers* (i.e., labels) contain the letters to compose a single word, assuming more than one pass could be required.

<br/>
<br/>
```raku
sub MAIN( *@strings is copy ) {
    my $letters = BagHash.new: @strings.pop.comb;
    my %stickers;
    my $loop = 0;

    while ( $letters.values.grep( * >= 1 ) ) {
	$loop++;

	for $letters.keys -> $needle {
	    next if ! $letters{ $needle };
	    my $found = False;
	    for @strings -> $word {
				if ( $word.comb.grep( $needle ) ) {
				    $letters{ $needle }--;
				    %stickers{ $word }{ $loop }.push: $needle;
				    $found = True;
				}
	    }

	    say "Cannot find $needle in any word!" and exit if ! $found;
	}
    }

    "$_ used { %stickers{ $_ }.keys.elems } times".say for %stickers.keys;
}

```
<br/>
<br/>


I classify the letters composing the initial word into a `Bag` (mutable), so that every time I find a letter in a sticker I can decrease the quantity and know how many letters remain to search for.
Every time I need to start over, I increase the looping counter, so that I can know how many times I need to use the same sticker over and over.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 216 - Task 1 - PL/Perl Implementation

Straightforward from the Raku implementation:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc216.task1_plperl( text, text[])
RETURNS SETOF text
AS $CODE$
   my ( $registration_code, $strings ) = @_;
   for my $word ( $strings->@* ) {
       my $matches = 0;
       for my $needle ( split( //, $word ) ) {
       	   $matches++ if ( grep( { $needle eq $_ } split( //, $registration_code ) ) );
       }

       return_next( $word ) if ( $matches == length( $word ) );
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

If I match every letter for a word, I can return it.


<a name="task2plperl"></a>
## PWC 216 - Task 2 - PL/Perl Implementation

Again, similar to the Raku implementation, but this time I return a table with a sticker, its run number and the letters extracted. Therefore, you can then query via SQL to get distinct stickers and other reporting data.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc216.task2_plperl( text, text[] )
RETURNS TABLE ( sticker text, run int, letter text )
AS $CODE$
   my ( $needle, $words ) = @_;
   my $searching_for = {};

   # create the bag
   for ( split //, $needle ) {
       $searching_for->{ $_ }++;
   }

   my $run = 0;
   while ( grep( { $_ >= 1 } values( $searching_for->%* ) ) ) {
   	 $run++;
         my $found = 0;

	 for my $letter ( keys $searching_for->%* ) {
	     next if ! $searching_for->{ $letter };
       	     for my $word ( $words->@* ) {
	     	 if ( grep( { $_ eq $letter } split( //, $word ) ) ) {
		    $searching_for->{ $letter }--;
		    return_next( { run => $run, sticker => $word, letter => $letter } );
		    $found++;
		    last;
		 }
	     }


	 }

	 if ( ! $found ) {
	    elog(INFO, "Cannot find match with letter $letter in any word!" );
	    return undef;
	 }
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 216 - Task 1 - PL/PgSQL Implementation

Here I join every word letters against the code, and see if there's a match that hit the length of the word itself.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc216.task1_plpgsql( rc text, strings text[] )
RETURNS SETOF TEXT
AS $CODE$
DECLARE
	current_word text;
	matches int;
BEGIN
	FOREACH current_word IN ARRAY strings LOOP
	    SELECT count(*)
	    INTO matches
	    FROM regexp_split_to_table( rc, '' ) r
	    , regexp_split_to_table( current_word, '' ) w
	    WHERE r = w;

	    IF matches = length( current_word ) THEN
	       RETURN NEXT current_word;
	    END IF;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 216 - Task 2 - PL/PgSQL Implementation

Here I use a temporary table as a bag, and update the table decreasing the number of letters extracted from the

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc216.task2_plpgsql( word text, stickers text[] )
RETURNS TABLE ( sticker text, run int, letter text )
AS $CODE$
DECLARE
	cl text;
	current_sticker text;
	m int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS letters( l text, c int DEFAULT 1, s text );
	TRUNCATE letters;

	INSERT INTO letters( l, c )
	SELECT ll, count(*)
	FROM regexp_split_to_table( word, '' ) ll
	GROUP BY ll;

	FOUND := true;
	run   := 0;
	WHILE FOUND LOOP
	      	run := run + 1;

		PERFORM count(*)
		FROM letters
		WHERE c > 0;

		IF NOT FOUND THEN
		   RETURN;
		END IF;

		FOR cl IN SELECT l FROM letters WHERE c > 0 LOOP
		    FOREACH current_sticker IN ARRAY stickers LOOP
		    	SELECT count(*)
				INTO m
				FROM regexp_split_to_table( current_sticker, '' ) s
				WHERE s = cl;


				IF m <= 0 THEN
				   CONTINUE;
				END IF;

			    UPDATE letters
				SET c = c - m
				   , s = s || ', ' || current_sticker;

				sticker := current_sticker;
				letter := cl;

				RETURN NEXT;
				EXIT; -- end this loop
		    END LOOP;
		END LOOP;
	END LOOP;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
