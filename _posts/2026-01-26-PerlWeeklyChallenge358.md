---
layout: post
title:  "Perl Weekly Challenge 358: using brute force!"
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

# Perl Weekly Challenge 358: using brute force!

This post presents my solutions to the [Perl Weekly Challenge 358](https://perlweeklychallenge.org/blog/perl-weekly-challenge-358/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 358 - Task 1 - Raku](#task1)
- [PWC 358 - Task 2 - Raku](#task2)
- [PWC 358 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 358 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 358 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 358 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 358 - Task 1 in PL/Java](#task1pljava)
- [PWC 358 - Task 2 in PL/Java](#task2pljava)
- [PWC 358 - Task 1 in Python](#task1python)
- [PWC 358 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 358 - Task 1 - Raku Implementation

The first task was about to find the *max string* in a given set. Each string is associated with a value that is its integer value or, in case it cannot be cast to an integer, its length.
A single line can solve the problem.

<br/>
<br/>
```raku
sub MAIN( *@strings ) {
    @strings.map( { $_ ~~ / ^ <[0 .. 9]>+ $ / ?? $_.Int !! $_.chars } ).max.say;
}

```
<br/>
<br/>

I `map` every string into its integer value, if the regular expression matches, or its length, then extra the `max` value.


<a name="task2"></a>
## PWC 358 - Task 2 - Raku Implementation

The second code was about trasnliterating a string with a given offset.


<br/>
<br/>
```raku
sub MAIN( Str $string, Int $value ) {
    my @alphabet = 'a' .. 'z';

    $string.lc.comb.map( -> $me { @alphabet[ ( $value + @alphabet.grep( * ~~ $me, :k ).first ) % @alphabet.elems ]  } ).join.say;

}

```
<br/>
<br/>

The idea is simple: every letter of the string (obtained via `comb`) is `map`ped to a letter of the `@aplphabet` at the position with the given offset, then `join`ed and printed out.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 358 - Task 1 - PL/Perl Implementation

Same approach as in Raku, but not having access to an array max value directly, I `sort` it reverse and return the first element.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc358.task1_plperl( text[] )
RETURNS int
AS $CODE$

   my ( $strings ) = @_;

   return (
	     sort { $b <=> $a }
	     map { $_ =~ / ^ \d+ $ /x ? int( $_ ) : length( $_ ) }
	     $strings->@*
	  )[ 0 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 358 - Task 2 - PL/Perl Implementation

Same approach as in Raku, but I need an inner function to get the index of a given letter in the array.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc358.task2_plperl( text, int )
RETURNS text
AS $CODE$

   my ( $string, $value ) = @_;

   my @alphabet = ( 'a' .. 'z' );

   my $index = sub {
        my ( $letter ) = @_;
	for my $i ( 0 .. $#alphabet - 1 ) {
	    return $i if ( $alphabet[ $i ] eq $letter );
	}

	return -1;
   };

   return
	join '',
	map { $alphabet[ ( $index->( $_ ) + $value ) % $#alphabet ] }
	split //, $string;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 358 - Task 1 - PL/PgSQL Implementation

I iterate over every string in the array, and compute its value, which is then directly compared with the max value found so far.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc358.task1_plpgsql( s text[] )
RETURNS int
AS $CODE$
DECLARE
	value int := 0;
	ok boolean := false;
	computed int := 0;
	cur text;
BEGIN

	FOREACH cur IN ARRAY s LOOP
		SELECT v IS NOT NULL
		INTO ok
		FROM regexp_matches( cur, '^\d+$' ) v;

		IF ok THEN
		   computed = cur::int;
		ELSE
		   computed = length( cur );
		END IF;

		IF computed > value THEN
		   value := computed;
		END IF;

	END LOOP;

	RETURN value;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 358 - Task 2 - PL/PgSQL Implementation

Here I cheat, and invoke PL/Perl directly.



<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc358.task2_plpgsql( s text, i int )
RETURNS text
AS $CODE$
   SELECT pwc358.task2_plperl( s, i );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 358 - Task 1 - PostgreSQL PL/Java Implementation

And here it comes the *brute force*: I `try` to convert every string into an integer, and in case of failure, I compute its `length`.
This approach is bovine and requires the setup of all the exception handling machine, which at runtime can be slower than its more cleaner counterpart testing the regular expression for a digit.


<br/>
<br/>
```java
    @Function( schema = "pwc358",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( String strings[] ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc358.task1_pljava" );

		int max = 0;

		for ( String s : strings ) {
		    int current = 0;
		    try {
				current = Integer.parseInt( s );
		    } catch( Exception e ) {
				current = s.length();
		    }

		    if ( current > max )
				max = current;
		}

		return max;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 358 - Task 2 - PostgreSQL PL/Java Implementation

Same approach as in other implementations: I compute the final index of every charater, find it in the alphabet and construct the final string one piece at a time.


<br/>
<br/>
```java
    @Function( schema = "pwc358",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task2_pljava( String src, int value ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc358.task2_pljava" );

		String alphabet = "abcdefghijklmnopqrstuvwxyz";
		String result   = "";

		for ( int i = 0; i < src.toLowerCase().length(); i++ ) {
		    char current = src.charAt( i );

		    int index = alphabet.indexOf( current );
		    index = ( index + value ) % alphabet.length();
		    current = alphabet.charAt( index );
		    result += current;
		}

		return result;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 358 - Task 1 - Python Implementation

Again, *brute force*: I `try` to convert every string into a number, and in case an `exception` is thrown, I use the `len`gth.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    max = 0
    for x in args:
        current = 0
        try:
            current = int( x )
        except:
            current = len( x )

        if current > max:
            max = current

    return max



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 358 - Task 2 - Python Implementation

Same implementation as in PL/Java, with the exception that `alphabet` is an array here.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    src      = args[ 0 ]
    value    = int( args[ 1 ] )
    result   = ""
    alphabet = list( 'abcdefghijklmnopqrstuvwxyz' )

    for c in src:
        index = ( value + alphabet.index( c ) ) % len( alphabet )
        result += alphabet[ index ]

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
