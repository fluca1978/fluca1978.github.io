---
layout: post
title:  "Perl Weekly Challenge 330: crunching words"
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

# Perl Weekly Challenge 330: crunching words

This post presents my solutions to the [Perl Weekly Challenge 330](https://perlweeklychallenge.org/blog/perl-weekly-challenge-330/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 330 - Task 1 - Raku](#task1)
- [PWC 330 - Task 2 - Raku](#task2)
- [PWC 330 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 330 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 330 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 330 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 330 - Task 1 in PL/Java](#task1pljava)
- [PWC 330 - Task 2 in PL/Java](#task2pljava)
- [PWC 330 - Task 1 in Python](#task1python)
- [PWC 330 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 330 - Task 1 - Raku Implementation

The first task was about to trim a given string so that each digit is removed according to the letter it has on the left.

<br/>
<br/>
```raku
sub MAIN( Str $string is copy
	  where { $string ~~ / ^ <[a .. z 0 .. 9 ]>+ $ / } ) {

    while ( $string ~~ / (<[a .. z]>) (<[ 0 .. 9 ]>) / ) {
		$string .= subst( / (<[a .. z]>) (<[ 0 .. 9 ]>) /, '' );
    }

    $string.say;
}

```
<br/>
<br/>

The idea is to loop while a regular expression that matches a letter followed by a number matches, and substitute the capture with nothing.


<a name="task2"></a>
## PWC 330 - Task 2 - Raku Implementation

Title case only words longer than three chars, and lower case all the rest.

<br/>
<br/>
```raku
sub MAIN( Str $sentence is copy  ) {
    my @parts;
    for $sentence.split( ' ' )  {
		@parts.push:  $_.Str.chars >= 3 ?? $_.tc !! $_.lc;
    }

    @parts.join( ' ' ).say;
}

```
<br/>
<br/>


I iterate over all the words and append the result, converted into `tc` (titlecase) or `lc` (lowercase) according to its length, so that I can re`join` the words.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 330 - Task 1 - PL/Perl Implementation

Very similar to the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc330.task1_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;
   die "Invalid string" unless( $string =~ / ^ [a-z0-9]+ $ /x );

   while ( $string =~ / [a-z] [0-9] /x ) {
   	 $string =~ s/ ([a-z]) ([0-9]) //x;
   }
   return $string;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 330 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, uses an array to keep track of the words and re-joins them at the end.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc330.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $text ) = @_;
   die "Invalid argument" unless( length( $text ) > 2 );

   my @parts;

   for ( split( ' ', $text ) ) {
       push @parts, length( $_ ) >= 3 ? ucfirst( $_ ) : lc( $_ );
   }

   return join( ' ', @parts );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 330 - Task 1 - PL/PgSQL Implementation

The same implementation as in Raku, except that the string `s` is substituted at every iteration.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc330.task1_plpgsql( s text )
RETURNS text
AS $CODE$
DECLARE

BEGIN
	WHILE regexp_match( s, '[a-z][0-9]' ) IS NOT NULL LOOP
	      s := regexp_replace( s, '([a-z])([0-9])', '' );
	END LOOP;

	RETURN s;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 330 - Task 2 - PL/PgSQL Implementation

Every word is cased appropriately, and then returned as part of the set of text to return.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc330.task2_plpgsql( s text )
RETURNS SETOF text
AS $CODE$
DECLARE
	current text;
BEGIN
	FOR current IN SELECT v FROM regexp_split_to_table( s, ' ' ) v LOOP
	    IF length( current ) >= 3 THEN
	       current := upper( current );
	    ELSE
	       current := lower( current );
	    END IF;

	    RETURN NEXT current;
	END LOOP;

RETURN;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 330 - Task 1 - PostgreSQL PL/Java Implementation

A not prefect solution: it iterates over the matches and substitutes them with nothing, then recompile the match and iterate again.

<br/>
<br/>
```java
    public static final String task1_pljava( String string ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc330.task1_pljava" );

		Pattern regexp = Pattern.compile( "([a-z])([0-9])" );
		Matcher engine = regexp.matcher( string );

		while ( engine.find() ) {
		    string = string.replaceAll( engine.group( 1 ) + engine.group( 2 ), "" );
		    engine = regexp.matcher( string );
		}

		return string;

    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 330 - Task 2 - PostgreSQL PL/Java Implementation

A Stream API based solution.

<br/>
<br/>
```java
    public static final String task2_pljava( String string ) throws SQLException {
	logger.log( Level.INFO, "Entering pwc330.task2_pljava" );


	return Arrays
	    .stream( string.split( " " ) )
	    .map( word -> word.length() <= 2
		  ? word.toLowerCase()
		  : Character.toTitleCase( word.charAt(0) ) + word.substring( 1 ).toLowerCase() )
	    .collect( Collectors.joining( " " ) );

    }
```
<br/>
<br/>

I use `stream` to `split` the string into words, then to 'map` into a variable `word` that is tested for its length and converted to the appropriate case, last all the stuff is `collect`ed with spaces.


# Python Implementations

<a name="task1python"></a>
## PWC 330 - Task 1 - Python Implementation

Basically, the same implementation as in other tasks.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    string = args[ 0 ]

    regexp = re.compile( r'[a-z][0-9]' )


    while regexp.search( string ) is not None:
        string = re.sub( r'[a-z][0-9]', '', string )

    return string

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 330 - Task 2 - Python Implementation

No regular expressions in here, since it does suffice to iterate over the whole list of arguments and case them according.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    parts = []

    for current in args :
        if len( current ) >= 3 :
            current = current.title()
        else:
            current = current.lower()

        parts.append( current )

    return ' '.join( parts )



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
