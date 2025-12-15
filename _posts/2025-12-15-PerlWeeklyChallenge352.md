---
layout: post
title:  "Perl Weekly Challenge 352: and here comes Christmas"
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

# Perl Weekly Challenge 352: and here comes Christmas

This post presents my solutions to the [Perl Weekly Challenge 352](https://perlweeklychallenge.org/blog/perl-weekly-challenge-352/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 352 - Task 1 - Raku](#task1)
- [PWC 352 - Task 2 - Raku](#task2)
- [PWC 352 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 352 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 352 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 352 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 352 - Task 1 in PL/Java](#task1pljava)
- [PWC 352 - Task 2 in PL/Java](#task2pljava)
- [PWC 352 - Task 1 in Python](#task1python)
- [PWC 352 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 352 - Task 1 - Raku Implementation

The task was about to find out if a word is contained in any subsequent words in a given list.

<br/>
<br/>
```raku
sub MAIN( *@strings ) {
    my %found;
    for 0 ..^ @strings.elems {
		my $current = @strings[ $_ ];
		%found{ $current } = @strings[ $_ + 1 .. * - 1 ].grep( * ~~ / $current / ).elems;
    }

    %found.kv.grep( -> $k, $v { $v > 0 } ).map( *[0] ).join( ', ' ).say;
}

```
<br/>
<br/>

This is an accurate implementation, since it uses a `%found` hash to count how many times the same word appears in the subsequent words.
The last line exploits the list of couples *keys, values* and searches for a value greater than zero, that is the word appears at least another time. The resulting list is then `map`ped to the key only, i.e., the word, and then the list is `join`ed and printed out.


<a name="task2"></a>
## PWC 352 - Task 2 - Raku Implementation

Given a list of bits, compose numbers from left to right one digit at a time, and see which are divisible by `5`.

<br/>
<br/>
```raku
sub MAIN( *@bits where { @bits.grep( * ~~ / <[0|1]> / ).elems == @bits.elems } ) {
    my @binaries;
    for 0 ..^ @bits.elems {
		my $val = @bits[ 0 .. $_ ].join.parse-base( 2 ).Int;
		@binaries.push: @bits[ 0 .. $_ ].join if ( $val %% 5 );
    }

    @binaries.join( ', ' ).say;
}

```
<br/>
<br/>

Simple enough, the `$val` is the joined bits from left to right, parsed in base `2` and converted to an integer. If such value is divisible `%%` by `5` the value is pushed to `@binaries`, that is last printed out.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 352 - Task 1 - PL/Perl Implementation

In this implementation, I output one at a time, the matching words by means of using a regular expression.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc352.task1_plperl( text[] )
RETURNS SETOF text
AS $CODE$

   my ( $strings ) = @_;

   for my $i ( 0 .. scalar $strings->@* ) {
     my $current = $strings->[ $i ];

     for my $j ( $i + 1 .. scalar $strings->@* ) {
     	 return_next( $current ) if ( $strings->[ $j ] =~ / $current /x );
     }
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 352 - Task 2 - PL/Perl Implementation

The implementation is pretty much the same as in Raku, using `oct` to convert the string of the binary bits into an integer.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc352.task2_plperl( int[] )
RETURNS SETOF text
AS $CODE$

   my ( $bits ) = @_;

   my @binaries;
   for ( 0 .. $bits->@* - 1 ) {
       push @binaries, join( '', $bits->@[ 0 .. $_ ] );
   }

   for my $val ( @binaries ) {
       if ( oct( "0b". $val ) % 5 == 0 ) {
       	  return_next( $val );
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
## PWC 352 - Task 1 - PL/PgSQL Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc352.task1_plpgsql( strings text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	counter int;
BEGIN
	counter := 0;

	FOR i IN 1 .. array_length( strings, 1 ) LOOP
	    SELECT count( v )
	    INTO   counter
	    FROM   unnest( strings[ i + 1 : array_length( strings, 1 ) ] ) v
	    WHERE  v like '%' || strings[ i ] || '%'
	    ;


	    IF counter > 0 THEN
	       RETURN NEXT strings[ i ];
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
## PWC 352 - Task 2 - PL/PgSQL Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc352.task2_plpgsql( bits bit[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	c int;
BEGIN

	FOR i IN 1 .. array_length( bits, 1 ) LOOP
	    c := cast( '0b' || array_to_string( bits[ 1 : i ], '' ) as int );
	    IF mod( c, 5 ) = 0 THEN
	       RETURN NEXT array_to_string( bits[ 1 : i ], '' );
	    END IF;
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
## PWC 352 - Task 1 - PostgreSQL PL/Java Implementation

Similar implementation to PL/Perl, using the regular expression engine to match against words.


<br/>
<br/>
```java
    @Function( schema = "pwc352",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String[] task1_pljava( String strings[] ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc352.task1_pljava" );

		List<String> found = new LinkedList<String>();

		for ( int i = 0; i < strings.length; i++ ) {
		    String current = strings[ i ];

		    Pattern regexp = Pattern.compile( current );

		    for ( int j = i + 1; j < strings.length; j++ ) {
				if ( regexp.matcher( strings[ j ] ).find() ) {
					found.add( current );
                }
		    }
		}


	return found.toArray( new String[0] );
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 352 - Task 2 - PostgreSQL PL/Java Implementation

First of all, I compose the `current` string by concatenating the bits. Then I convert it into an integer, and check if it is a multiple of `5`, in such case I add the value to the list `found` that is then returned.


<br/>
<br/>
```java
    @Function( schema = "pwc352",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String[]  task2_pljava( String[] bits ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc352.task2_pljava" );

		List<String> found = new LinkedList<String>();

		for ( int i = 0; i < bits.length; i++ ) {
		    String current = "0";

		    for ( int j = 0; j <= i; j++ ) {
				current += bits[ j ];
		    }

		    int val = Integer.parseInt( current, 2 );
		    if ( val % 5 == 0 )
				found.add( current );
		}

		return found.toArray( new String[0] );

    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 352 - Task 1 - Python Implementation

Similar to the PL/Perl implementation.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    found = []

    for i in range( 0, len( args ) ) :
        current = args[ i ]
        for j in range( i + 1, len( args ) ) :
            if re.match( current, args[ j ] ) :
                found.append( current )

    return found


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 352 - Task 2 - Python Implementation

Same implementation as in PL/Java, but please note how verbose it is to pass from a list to a `map`ped list.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    bits = list( map( int, args ) )
    found = []

    for i in range( 0, len( bits ) ) :
        current = ''.join( list( map( str, bits[ 0 : i + 1 ] ) ) )
        if int( current, 2 ) % 5 == 0 :
            found.append( current )

    return found


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
