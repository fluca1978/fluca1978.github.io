---
layout: post
title:  "Perl Weekly Challenge 190: decoding strings"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 190: decoding strings

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 190](https://perlweeklychallenge.org/blog/perl-weekly-challenge-190/){:target="_blank"}.

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
## PWC 190 - Task 1 - Raku Implementation

Given a word as input argument, check if the word has either (i) a single capital letter at the very beginning, or (ii) all capital letters or (iii) all lowercase letters.
This is quite simple to check with a single regular expression.

<br/>
<br/>
```raku
sub MAIN( Str $word ) {

    '1'.say and exit if ( $word ~~ /
			  | ^ <[A .. Z]> <[a .. z]>+ $
			  |  ^ <[a..z]>+ $
			  | ^ <[A..Z]>+ $ / );
    '0'.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 190 - Task 2 - Raku Implementation

Given a number-only string, convert any digit to a letter and output the resulting decoded string.

<br/>
<br/>
```raku
sub MAIN( Str $number where { $number ~~ / ^ \d+ $ / } ) {
    my %decode-table = ( 0 .. 9 ).map: { ( $_ + 1 ) => ( 'A' .. 'Z' )[ $_ ]  };
    my @decoded;
    for $number.comb -> $current {
   	   @decoded.push: %decode-table{ $current };
    }

    @decoded.join( '' ).say;
}

```
<br/>
<br/>

I decided to create a `%decode-tabnle` hash that contains, given any possible digit, the corresponding letter. To achieve this, I do `map` the list of numbers creating a set of pairs, keyed with the numeric value and with a value corresponding to the letter.
<br/>
Then I iterate on the single digits, by means of `comb` and push every letter into a `@decoded` array. At last, I do print the resulting array as a single string.



<a name="task1plperl"></a>
## PWC 190 - Task 1 - PL/Perl Implementation

A simple solution that reflect the Raku implementation: I check the argument `$word` against one of the regular expression that will match the case required. If a match is found, I do quickly return from the function.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc190.task1_plperl( text )
RETURNS int
AS $CODE$
 my ( $word ) = @_;
 return 1 if ( $word =~ / ^ [A-Z] [a-z]+ $ /x
             || $word =~ / ^ [a-z]+ $ /x
	     || $word =~ / ^ [A-Z]+ $ /x );

 return 0;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 190 - Task 2 - PL/Perl Implementation

Same implementation as in Raku: I do create an hash out of the mapping between letters and digits, than walk thru every single digit of the input string and build a `@decoded` array, that is then `join`ed to produce the resulting string.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc190.task2_plperl( text )
RETURNS text
AS $CODE$
 my ( $number ) = @_;
 my @decoded;
 my %decode_table = map { ( $_ + 1 ) => ( 'A' .. 'Z' )[ $_ ] } ( 0 .. 9 );
 for my $current ( split '', $number ) {
     push @decoded, $decode_table{ $current };
 }

return join( '', @decoded );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task1plpgsql"></a>
## PWC 190 - Task 1 - PL/PgSQL Implementation

The trick is to match the input string against a regular expression, and this can be done with the `~` operator. Please note that the regular expression is *text* and not a first class object.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc190.task1_plpgsql( word text )
RETURNS int
AS $CODE$
BEGIN
	IF word ~ '^[A-Z][a-z]+$' THEN
	   RETURN 1;
	ELSIF word ~ '^[a-z]+$' THEN
	   RETURN 1;
	ELSIF word ~ '^[A-Z]+$' THEN
	   RETURN 1;
	ELSE
	  RETURN 0;
	END IF;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 190 - Task 2 - PL/PgSQL Implementation

This task can be solved with a more SQL-like approach.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc190.task2_plpgsql( num text)
RETURNS SETOF text
AS $CODE$
DECLARE
	decoded text := '';
BEGIN
	CREATE TEMP TABLE IF NOT EXISTS decode_table ( c char, i int );
	TRUNCATE TABLE decode_table;
	INSERT INTO decode_table
	VALUES
	  ( 'A', 0 )
	, ( 'B', 1 )
	, ( 'C', 2 )
	, ( 'D', 3 )
	, ( 'E', 4 )
	, ( 'F', 5 )
	, ( 'G', 6 )
	, ( 'H', 7 )
	, ( 'I', 8 )
	, ( 'M', 9 );

	RETURN QUERY
	WITH w AS ( SELECT n::int, row_number() over () as r FROM regexp_split_to_table( num, '' ) n )
	, dec AS (
		SELECT d.c
		FROM decode_table d
		JOIN w ON w.n = d.i
		ORDER BY w.r
	)
	SELECT string_agg( dec.c, '' )
	FROM dec;



END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

First of all, the function creates a temporary table, if not already there, and pushes the corresponding letters and numbers into it. Please note that the table is truncated, because in the case of subsequent calls we don't want to insert other values into it.
<br/>
Then I build a query made of three parts:
- `w` is a table that contains every single digit from the incoming string in a single row. Note that I add also a *row number* because I need to keep the input order;
- `dec` is a table that contains a character decoded from the temporary table. Such table is ordered by the row number at the previous step, so the decoded letters are in the correct order;
- the main `SELECT` does a `string_agg` to compose every row from the `dec` table into a single string, producing the correct output.
