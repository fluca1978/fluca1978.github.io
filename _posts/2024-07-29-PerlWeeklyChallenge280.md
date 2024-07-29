---
layout: post
title:  "Perl Weekly Challenge 280: grepping everything!"
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

# Perl Weekly Challenge 280: grepping everything!

This post presents my solutions to the [Perl Weekly Challenge 280](https://perlweeklychallenge.org/blog/perl-weekly-challenge-280/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 280 - Task 1 - Raku](#task1)
- [PWC 280 - Task 2 - Raku](#task2)
- [PWC 280 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 280 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 280 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 280 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 280 - Task 1 in PL/Java](#task1pljava)
- [PWC 280 - Task 2 in PL/Java](#task2pljava)
- [PWC 280 - Task 1 in Python](#task1python)
- [PWC 280 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 280 - Task 1 - Raku Implementation

The first task was about finding out the first (i.e., leftmost) letter that appears in a given string that has other repetitions (i.e., occurrencies) in the given string itself.

<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / ^ <[a..z]>+ $ / } ) {
    for $string.comb -> $needle {
		$needle.say and exit if ( $string.comb.grep( * ~~ $needle ).elems > 1 );
    }
}

```
<br/>
<br/>

The idea is to split the string into its chars, by means of `comb`, and then to `grep` again the whole string for the `$needle` current char. If the counting has more than one element (because the char matches itself, obviously), I've found the char so there is no need to continue: I print the char and exit.



<a name="task2"></a>
## PWC 280 - Task 2 - Raku Implementation

Given a string where the pipe `|` separates keys from values into pairs, count the asterisks that appear in every key and report the sum.

<br/>
<br/>
```raku
sub MAIN( Str $input is copy where { $input ~~ / <[|]>+ / } ) {

    # avoid the last bar to trigger an empty pair
    if $input ~~ / <[|]> $ / {
		$input = $input.subst( / <[|]> $ /, '' );
    }

    my @pairs;
    for $input.split( '|' ) -> $k, $v {
		next if ! $k || ! $v;
		@pairs.push: $k => $v;
    }

    @pairs.map( { $_.key.comb.grep( * ~~ '*' ).elems } ).sum.say;

}

```
<br/>
<br/>

First of all, if the string ends with the pipe, remove it, so that there are no "empty" pairs.
The put all the pairs into the `@pairs` array as `Pair` objects by means of the fat comma.
Last, `map` the `@pairs` into the count of the asteriks within each key, and `sum` the result.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 280 - Task 1 - PL/Perl Implementation

The idea is simple: create an array of `@chars` from the input string, and evaluate every `$needl` char.
If the `$needle` appears more than one time in the array of chars I return the `$needle` and stop the execution.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc280.task1_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;
   my @chars = split //, $string;

   for my $needle ( @chars ) {
       return $needle if ( grep( { $_ eq $needle } @chars ) > 1 );
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 280 - Task 2 - PL/Perl Implementation

Here I take another approach with regard to the Raku implementation: I build an array of `@parts` and keep a `$counter` to count the asterisks in a cumulative way. Then, I iterate over the `@parts` and skip all those elements that are not keys (i.e., have an even index), and then count the asterisks.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc280.task2_plperl( text )
RETURNS int
AS $CODE$

   my ( $input ) = @_;
   my @pairs;

   my @parts = split /[|]/, $input;
   my $counter = 0;
   for ( 0 .. @parts - 1 ) {
       next if $_ % 2; # take only left side of a pair
       my $needle = $parts[ $_ ];

       next if ! $needle;
       $counter += scalar grep { $_ eq '*' } split //, $needle;
   }

   return $counter;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 280 - Task 1 - PL/PgSQL Implementation

A loop on the splitted string helps implementing the same alghoritm as in PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc280.task1_plpgsql( s text )
RETURNS char
AS $CODE$
DECLARE
	needle char;
	counting int;
BEGIN

	FOR needle IN SELECT v::char FROM regexp_split_to_table( s, '' ) v LOOP
		counting := 0;
		SELECT count( * )
		INTO counting
		FROM regexp_split_to_table( s, '' ) v
		WHERE v = needle;

		IF counting > 1 THEN
		   RETURN needle;
		END IF;
	END LOOP;

	RETURN NULL;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 280 - Task 2 - PL/PgSQL Implementation

A single query with CTE can do the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc280.task2_plpgsql( s text )
RETURNS int
AS $CODE$

   WITH searching_for AS (
   	SELECT v::text, row_number() over () as r
   	FROM regexp_split_to_table( s, '[|]' ) v
   )
   , data_parts AS (
   	      SELECT v
	      FROM searching_for
	      WHERE r % 2 <> 0
   )
   SELECT sum( length( v ) - length( replace( v, '*', '' ) ) )
   FROM data_parts;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The `searching_for` materializes each letter with its order, then `data_parts` considers only the keys (i.e, parts with an odd ordering) and last the `SELECT` computes the number of asterisk with an awful trick: counting the length of the string minus the length of the string having removed the asteriks.


# Java Implementations

<a name="task1pljava"></a>
## PWC 280 - Task 1 - PostgreSQL PL/Java Implementation

Nothing fancy here: just iterate over the string splitted into letters.
<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc280",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static String  task1_pljava( String string ) throws SQLException {
	logger.log( Level.INFO, "Entering pwc280.task1_pljava" );

		for ( String needle : string.split( "" ) ) {
		    int count = 0;
		    for ( String s : string.split( "" ) )
				if ( s.equals( needle ) )
				    count++;


			    if ( count > 1 )
					return needle;
		}

		return null;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 280 - Task 2 - PostgreSQL PL/Java Implementation

Again, nothing fancy, no streams, no magic, simple nested loops to implement something similar to PL/Perl.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc280",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( String input ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc280.task2_pljava" );


		int pos = -1;
		int count = 0;
		for ( String needle : input.split( "[|]" ) ) {
		    pos++;
		    if ( pos % 2 != 0 )
				continue;

		    // here I've a left part in the pair
		    for ( String a : needle.split( "" ) )
				if ( a.equals( "*" ) )
					count++;
		}

		return count;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 280 - Task 1 - Python Implementation

Really simple to implement, since Python allows for a quick iteration over string chars.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    string = args[ 0 ]
    for l in string:
        if len( list( filter( lambda x: x == l, string ) ) ) > 1:
            return l


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 280 - Task 2 - Python Implementation

Similar approach to PL/Perl, but with an other trick.
I iterate over every char, and when I find a pipe, I increment the `pos` indicator. If I encounter an asterisk, and the `pos` indicates that I'm analyzing a key, I increment the counter. Therefore, there is no need for intermediate data transformation.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    pos = 0
    count = 0
    for needle in args[ 0 ]:
        if needle == '|':
            pos += 1
            continue
        elif needle == '*' and pos % 2 != 0:
            count += 1

    return count

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
