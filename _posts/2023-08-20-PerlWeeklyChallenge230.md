---
layout: post
title:  "Perl Weekly Challenge 230: coming back from the mountains!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 230: coming back from the mountains!

This post presents my solutions to the [Perl Weekly Challenge 230](https://perlweeklychallenge.org/blog/perl-weekly-challenge-230/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 230 - Task 1 - Raku](#task1)
- [PWC 230 - Task 2 - Raku](#task2)
- [PWC 230 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 230 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 230 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 230 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 230 - Task 1 - Raku Implementation

The first task asked to seprate every incoming number into its digits.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my @digits;
    @digits.push( | $_.comb ) for @nums;
    @digits.join( ', ' ).say;
}

```
<br/>
<br/>

The only note here is that I need to flatten the list coming out from `comb` in order to not get nested arrays.


<a name="task2"></a>
## PWC 230 - Task 2 - Raku Implementation

The second task was about finding out which given words begin with a given prefix. Sounds like an easy job for the regular expression engine.

<br/>
<br/>
```raku
sub MAIN( Str $prefix where { $prefix.chars >= 2 },
	  *@words where { @words.elems == @words.grep( * ~~ Str ).elems } ) {
    my $count = 0;
    $count++ if ( $_ ~~ / ^ $prefix / ) for @words;
    $count.say;

}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 230 - Task 1 - PL/Perl Implementation

Using `split` it is easy to get the list of digits.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc230.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   for ( $nums->@* ) {
       for ( split //, $_ ) {
       	   return_next( $_ );
       }
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 230 - Task 2 - PL/Perl Implementation

Another quick and easy job for a regexp!

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc230.task2_plperl( text, text[] )
RETURNS int
AS $CODE$
   my ( $prefix, $words ) = @_;
   my $count = 0;

   for ( $words->@* ) {
       $count++ if ( $_ =~ / ^ $prefix /x );
   }

   return $count;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 230 - Task 1 - PL/PgSQL Implementation

A quite straightforward approach to extract the digits into a sub array and return each of them,

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc230.task1_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	n int;
	nn text;
BEGIN
	FOREACH n IN ARRAY nums LOOP
		FOREACH nn IN ARRAY regexp_split_to_array( n::text, '' ) LOOP
			RETURN NEXT nn::int;
		END LOOP;
	END LOOP;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 230 - Task 2 - PL/PgSQL Implementation

Regexp engine here, but checking if returns at least a result when asking for a dynamicallt built regular expression.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc230.task2_plpgsql( pref text, words text[] )
RETURNS INT
AS $CODE$
DECLARE
	current_word text;
	counter int := 0;
BEGIN
	FOREACH current_word IN ARRAY words LOOP
		IF regexp_match( current_word, '^' || pref ) IS NOT NULL THEN
		   counter := counter + 1;
		END IF;
	END LOOP;

	RETURN counter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
