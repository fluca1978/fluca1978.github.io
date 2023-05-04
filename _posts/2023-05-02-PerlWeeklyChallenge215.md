---
layout: post
title:  "Perl Weekly Challenge 215: I skipped the previous week, but I'm back on track!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 215: I skipped the previous week, but I'm back on track!

This post presents my solutions to the [Perl Weekly Challenge 215](https://perlweeklychallenge.org/blog/perl-weekly-challenge-215/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 215 - Task 1 - Raku](#task1)
- [PWC 215 - Task 2 - Raku](#task2)
- [PWC 215 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 215 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 215 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 215 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 215 - Task 1 - Raku Implementation

The first task was about receiving a list of words and printing out how many are unosrted, i.e., do not have letters sorted within it.

<br/>
<br/>
```raku
sub MAIN( *@words where { @words.grep( * ~~ / ^ <[a..zA..Z]>+ $ / ).elems == @words.elems } ) {

    say ( @words.elems -  @words.grep( { $_ ~~ $_.comb.sort.join } ).elems );
}

```
<br/>
<br/>

*One line of code!*
<br/>
The idea is to *smart match* every word in `@words` with its counterpart list of letters obtained thru `comb` and `sort`ed, keeping only thos `@words` that are equal to their sorted counterpart. Therefore, making a difference between the number of elements provides the result.

<a name="task2"></a>
## PWC 215 - Task 2 - Raku Implementation

Given a string of bits and an integer value, try to change all the specified values from `0` to `1` only if the zero that is going to change is surrounded by other zeros.

<br/>
<br/>
```raku
sub MAIN( :$count is copy where { $count >= 0 } ,
		*@digits is copy where { @digits.grep( 0 <= *.Int <= 1  ).elems == @digits.elems }  ) {

    my $done = False;
    while $count {
	$done = False;
	for 1 ..^ @digits.elems - 1 {
	    if ( @digits[ $_ ] == 0 && @digits[ $_ - 1 ] == 0 && @digits[ $_ + 1 ] == 0 ) {
				@digits[ $_ ] = 1;
				$count--;
				$done = True;
				last;
	    }
	}

	'0'.say and exit if ! $done;
    }

    '1'.say and exit if ! $count;
    '0'.say;
}

```
<br/>
<br/>


# PL/Perl Implementations



<a name="task1plperl"></a>
## PWC 215 - Task 1 - PL/Perl Implementation

The same approach as Raku, clearly with operators that read from left to right:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc215.task1_plperl( text[] )
RETURNS SETOF text
AS $CODE$
  my ( $words ) = @_;
  for ( $words->@* ) {
      return_next( $_ ) if ( $_ eq join( '', sort( split( //, $_ ) ) ) );
  }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The function returns the list of words that are sorted, so that it is possible to count them. This is a mistake in the implementation, but I think it is more useful to return the result set of words rather than the number.


<a name="task2plperl"></a>
## PWC 215 - Task 2 - PL/Perl Implementation

Same approach as in the Raku solution:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc215.task2_plperl( int, int[] )
RETURNS int
AS $CODE$
   my ( $count, $digits ) = @_;
   my ( $done );

   while ( $count ) {
   	 $done = 0;
	 for ( 1 .. scalar( $digits->@* ) - 1 ) {
	     if ( $digits->[ $_ ] == 0
	     	&& $digits->[ $_ - 1 ] == 0
		&& $digits->[ $_ + 1 ] == 0 ) {
		$digits->[ $_ ] = 1;
		$done = 1;
		$count--;
		last;
	     }
	 }

	 last if ! $done;
   }

   return 1 if ! $count;
   return 0;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 215 - Task 1 - PL/PgSQL Implementation

Same approach as PL/Perl:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc215.task1_plpgsql( words text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	current_word text;
	sorted_word  text;
	current_letter text;
BEGIN
	FOREACH current_word IN ARRAY words LOOP
	        /*
		sorted_word := '';
		FOR current_letter IN SELECT l FROM regexp_split_to_table( current_word, '' ) l ORDER BY 1 LOOP
		    sorted_word := sorted_word || current_letter;
   	        END LOOP;
                */

		SELECT string_agg( c, '' )
		INTO sorted_word
		FROM ( SELECT c
		       FROM regexp_split_to_table( current_word, '' ) c
		       ORDER BY 1
		     ) c;

		IF sorted_word = current_word THEN
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


In the beginning I did a *foreach* loop over the array of letters, sorted, so to compose the sorted word.
Then I thought a little more, and I decided to do the same with a subquery:
- I split the word into letters with `regexp_split_to_table`;
- I order by each letter, so to get a sorted list of letters in the form of a table;
- I select every letter out of the above subquery;
- I compose the final word with the `string_agg` function.


<a name="task2plpgsql"></a>
## PWC 215 - Task 2 - PL/PgSQL Implementation

Same approach as in PL/Perl, but this time returning a boolean value:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc215.task2_plpgsql( c int, digits int[] )
RETURNS boolean
AS $CODE$
DECLARE
	done boolean;
	i    int;
BEGIN
	i := 2;
	WHILE c > 0 LOOP
	      done := false;

	     WHILE i < array_length( digits, 1 )  LOOP
	     	 IF digits[ i ] = 0 AND digits[ i - 1 ] = 0 AND  digits[ i + 1 ] = 0 THEN
		    digits[ i ] := 1;
		    done := true;
		    c := c - 1;
		    EXIT;
		 END IF;

		 i := i + 1;
	     END LOOP;


	     IF NOT done THEN
	     	EXIT;
	     END IF;
	END LOOP;

	IF c = 0 THEN
	   RETURN true;
	ELSE
	   RETURN false;
	END IF;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
