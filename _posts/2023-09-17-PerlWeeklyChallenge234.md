---
layout: post
title:  "Perl Weekly Challenge 234: nested loops"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 234: nested loops

This post presents my solutions to the [Perl Weekly Challenge 234](https://perlweeklychallenge.org/blog/perl-weekly-challenge-234/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 234 - Task 1 - Raku](#task1)
- [PWC 234 - Task 2 - Raku](#task2)
- [PWC 234 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 234 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 234 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 234 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 234 - Task 1 - Raku Implementation

The first task was about finding out all characters that appear in a list of words.

<br/>
<br/>
```raku
sub MAIN( *@words ) {
    my %chars;
    for @words -> $current_word {
		for $current_word.comb.unique.sort {
		    %chars{ $_ }.push: $current_word;
		}
    }

    for %chars.kv -> $char, $list {
		say $char if $list.elems == @words.elems;
    }

}

```
<br/>
<br/>


I first disassemble each word, and then print out those chars that have an *appearing list* of words of the same size of the input.


<a name="task2"></a>
## PWC 234 - Task 2 - Raku Implementation

The second taskw as about finding a triplet of array indexes so that each number is different from the other twos. I used a nested loop do achieve the aim.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    my @triples;

    #(i, j, k) that satisfies num[i] != num[j], num[j] != num[k] and num[k] != num[i].
    for 0 ..^ @nums.elems - 2  -> $i {
		for $i ^..^ @nums.elems - 1 -> $j {
		    for $j ^..^ @nums.elems -> $k {
						@triples.push: [ $i, $j, $k ] if ( @nums[ $i ] != @nums[ $j ]
										   && @nums[ $j ] != @nums[ $k ]
										   && @nums[ $k ] != @nums[ $i ] );
		    }
		}
    }

    @triples.join( "\n" ).say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 234 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, using an hash to keep track of the charaters and `grep` to search fo duplicated words.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc234.task1_plperl( text[] )
RETURNS SETOF char
AS $CODE$
  my ( $words ) = @_;

  my $chars = {};

  for my $current_word ( $words->@* ) {
      for my $current_char ( sort split //, $current_word ) {
            if ( ! grep { $_ eq $current_word } $chars->{ $current_char }->@* ) {
	     push $chars->{ $current_char }->@*, $current_word;
	  }
      }
  }

  for my $current_char ( keys $chars->%* ) {
      return_next( $current_char ) if ( $chars->{ $current_char }->@* == scalar $words->@* );
  }

  return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 234 - Task 2 - PL/Perl Implementation

A nested loop over the arrays to find the triplets. Note that the function returns a `table`, so an hash is returned.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc234.task2_plperl( int[] )
RETURNS TABLE( i int, j int, k int )
AS $CODE$
   my ( $nums ) = @_;

   for my $i ( 0 .. $nums->@* - 3 ) {
       for my $j ( $i + 1 .. $nums->@* - 2 ) {
       	   for my $k ( $j + 1 .. $nums->@* - 1 ) {
	       return_next( { i => $i, j => $j, k => $k  } )
	       	       if ( $nums->[ $i ] != $nums->[ $j ]
	       	       	  && $nums->[ $j ] != $nums->[ $k ]
			  && $nums->[ $k ] != $nums->[ $i ] );
	   }
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
## PWC 234 - Task 1 - PL/PgSQL Implementation

This is not an optimal solution, but it is the only one that comes into my mind right now.
I use a temporary table to store the chars and the words they belong to, then extract all the chars with more occurrencies than the words.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc234.task1_plpgsql( words text[] )
RETURNS SETOF char
AS $CODE$
DECLARE
	w text;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS chars( c char, word text );
	TRUNCATE chars;

	FOREACH w IN ARRAY words LOOP
		INSERT INTO chars
		SELECT c, w
		FROM regexp_split_to_table( w, '' ) c;
	END LOOP;

	RETURN QUERY
	       SELECT c
	       FROM chars
	       GROUP BY c
	       HAVING count( word ) >= array_length( words, 1 );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 234 - Task 2 - PL/PgSQL Implementation

Nested loops, nothing to say here.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc234.task2_plpgsql( nums int[] )
RETURNS TABLE (ii int, jj int, kk int )
AS $CODE$
BEGIN
	FOR i IN 1 .. array_length( nums, 1 ) - 2 LOOP
	    FOR j IN i + 1 .. array_length( nums, 1 ) - 1 LOOP
	    	FOR k IN j + 1 .. array_length( nums, 1 ) LOOP
		    IF nums[i] <> nums[j] AND nums[j] <> nums[k] AND nums[k] <> nums[i] THEN
		       raise info '% % %', i, j, k;
		       ii := i;
		       jj := j;
		       kk := k;
		       RETURN NEXT;
		    END IF;
		END LOOP;
	    END LOOP;
	END LOOP;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
