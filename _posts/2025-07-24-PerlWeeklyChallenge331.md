---
layout: post
title:  "Perl Weekly Challenge 331: String-ish"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 331: String-ish

This post presents my solutions to the [Perl Weekly Challenge 331](https://perlweeklychallenge.org/blog/perl-weekly-challenge-331/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 331 - Task 1 - Raku](#task1)
- [PWC 331 - Task 2 - Raku](#task2)
- [PWC 331 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 331 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 331 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 331 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 331 - Task 1 in PL/Java](#task1pljava)
- [PWC 331 - Task 2 in PL/Java](#task2pljava)
- [PWC 331 - Task 1 in Python](#task1python)
- [PWC 331 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 331 - Task 1 - Raku Implementation

The first task was about computing the length of the last word in a sentence, skipping spaces.

<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    $string.split( /\s+/ ).grep( *.chars > 0 )[ * - 1 ].chars.say;
}

```
<br/>
<br/>

A single line does suffice: the `$string` is `split` by spaces, and only words with a defined length is kept. Then I took the last element in the list of words and compute the `chars` length.



<a name="task2"></a>
## PWC 331 - Task 2 - Raku Implementation

Given two string, see if flipping only one char can make the strings the same.


<br/>
<br/>
```raku
sub MAIN( Str $left, Str $right ) {
    True.say and exit if ( $left.fc ~~ $right.fc );
    False.say and exit if ( $left.chars != $right.chars );

    my $permutations = +( [Z] $left.comb, $right.comb ).grep( { $_[ 0 ] ne $_[ 1 ] } );
    False.say and exit if ( $permutations > 2 );
    True.say;

}

```
<br/>
<br/>

There is much more code to do some initial checks than to performe the whole logic.
First of all, if the words do not have the same length or are already the same, I can stop the application.

Otherwise, I `[Z]` zip the array of chars, so that I get a couple of chars for every couple of letters, and then `grep` the arrays that have different letters. If the `$permutations` counts more than `2` changes, then flipping a single letter (i.e., two differences) does not suffice.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 331 - Task 1 - PL/Perl Implementation

A single line to do all.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc331.task1_plperl( text )
RETURNS int
AS $CODE$

   my ( $sentence ) = @_;

   return (
      map { length $_ }
      split( /\s+/,  $sentence ) )[ - 1 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I `split` the `$sentence` by spaces, `map`ping the length of the resulting words and getting the last element of the array of lengths.


<a name="task2plperl"></a>
## PWC 331 - Task 2 - PL/Perl Implementation

Similar approach to Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc331.task2_plperl( text, text )
RETURNS boolean
AS $CODE$

   my ( $left, $right ) = @_;

   return 1 if ( lc( $left ) eq lc( $right ) );
   return 0 if ( length( $left ) != length( $right ) );

   my $index = 0;
   my @l = split //, $left;
   my @r = split //, $right;

   my @diff =
       grep { $_->[ 0 ] ne $_->[ 1 ] }
       map { [ $l[ $index ], $r[ $index++ ] ] }
       ( 0 .. $#l );

   return $#diff < 2 ? 1 : 0;


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I create two arrays of letters by means of `split`, then `map` every letter into an array of two elements, `grep`ping the subarrays that have differences and counting in the end.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 331 - Task 1 - PL/PgSQL Implementation

A single query does the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc331.task1_plpgsql( s text )
RETURNS int
AS $CODE$

   SELECT l
   FROM (
	SELECT length( v::text ) as l, row_number() over () as r
	FROM regexp_split_to_table( s, '\s+' ) v
	WHERE length( v::text ) > 0
	ORDER BY r DESC
	)
   LIMIT 1;


$CODE$
LANGUAGE sql;

```
<br/>
<br/>

In the inner query I compute the length of every word by means of splitting the sentence with a regular expression, and getting the last word by ordering the result by the row number. Then I select the length of the very first line.


<a name="task2plpgsql"></a>
## PWC 331 - Task 2 - PL/PgSQL Implementation

Similar approach to PL/Perl one.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc331.task2_plpgsql( l text, r text)
RETURNS boolean
AS $CODE$
DECLARE
	diffs int := 0;
BEGIN
	IF l = r THEN
	   RETURN true;
	END IF;

	IF length( l ) != length( r ) THEN
	   RETURN false;
	END IF;

	WITH
	lt AS (
	     SELECT lc::text, row_number() over () as lr
	     FROM regexp_split_to_table( l, '' ) lc
	)
	, rt AS (
	     SELECT rc::text, row_number() over () as rr
	     FROM regexp_split_to_table( r, '' ) rc
	)
	SELECT count(*)
	into diffs
	FROM lt, rt
	WHERE rr = lr
	AND lc <> rc
	;

	IF diffs > 2 THEN
	   RETURN false;
	ELSE
	   RETURN true;
	END IF;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


With a CTE I define two tables with a character and the row number on every row, then join the tables on the same row number and count how many differences in chars there are.

# Java Implementations

<a name="task1pljava"></a>
## PWC 331 - Task 1 - PostgreSQL PL/Java Implementation

Iterate on every word, and keep track of the length of the current word.
Once the iteration ends, the result is the length of the last word.

<br/>
<br/>
```java
    public static int task1_pljava( String sentence ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc331.task1_pljava" );

		int length = 0;

		for ( String s : sentence.split( "\\s+" ) ) {
		    s = s.trim();
		    if ( s.length() == 0 )
				continue;

		    length = s.length();
		}

		return length;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 331 - Task 2 - PostgreSQL PL/Java Implementation

Same implementation of PL/Perl: iterate over each char of the two words and count the differences.


<br/>
<br/>
```java
    public static final boolean task2_pljava( String left, String right ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc331.task2_pljava" );

		if ( left.length() != right.length() )
		    return false;

		left = left.toLowerCase();
		right = right.toLowerCase();
		int diff = 0;

		for ( int i = 0; i < left.length(); i++ ) {
		    if ( left.charAt( i ) != right.charAt( i ) )
				diff++;
		}

		return diff <= 2;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 331 - Task 1 - Python Implementation

Same implementation of PL/Java: iterate over each word and keep track of the length, so that the last length computed is the length o the last word.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    size = 0
    for w in args[ 0 ].split() :
        if len( w ) > 0 :
            size = len( w )

    return size


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 331 - Task 2 - Python Implementation

Same implementation of PL/Java.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    left  = args[ 0 ].lower()
    right = args[ 1 ].lower()

    diffs = 0

    if len( left ) != len( right ) :
        return False


    for i in range( 0, len( left ) ) :
        if left[ i ] != right[ i ] :
            diffs += 1


    return diffs <= 2

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
