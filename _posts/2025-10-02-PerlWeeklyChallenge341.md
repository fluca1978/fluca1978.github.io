---
layout: post
title:  "Perl Weekly Challenge 341: back from Japan"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
I'm back from Japan!

# Perl Weekly Challenge 341: back from Japan

It has been a while since I committed something to the Weekly Challenge, and this is due to the fact
that **I spent the last weeks preparing, and most notably, enjoying, a trip to Japan** with my family.


This post presents my solutions to the [Perl Weekly Challenge 341](https://perlweeklychallenge.org/blog/perl-weekly-challenge-341/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 341 - Task 1 - Raku](#task1)
- [PWC 341 - Task 2 - Raku](#task2)
- [PWC 341 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 341 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 341 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 341 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 341 - Task 1 in PL/Java](#task1pljava)
- [PWC 341 - Task 2 in PL/Java](#task2pljava)
- [PWC 341 - Task 1 in Python](#task1python)
- [PWC 341 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 341 - Task 1 - Raku Implementation

The first task was about to remove all the words in a sentence that contain a given letter from a list.

<br/>
<br/>
```raku
sub MAIN( Str $text, *@keys ) {
    $text.say and exit( 0 ) unless ( @keys.elems );

    my @words;
    for $text.split( / \s+ / ) -> $current-word {
		my $found = 0;
		for @keys {
			$found += $current-word.comb.grep( * ~~ $_ ).elems;
		}
		last if $found;

	    @words.push: $current-word;
    }

    @words.join( ' ' ).say;
}

```
<br/>
<br/>

A nested loop solve the problem, splitting the sentence into words and then checking every word against a list of keys.


<a name="task2"></a>
## PWC 341 - Task 2 - Raku Implementation

The second task was about reversing the leftmost part of a given string up to a given letter, concatenating it to the remaining part of the string.


<br/>
<br/>
```raku
sub MAIN( Str $text, Str $prefix where { $text ~~ / $prefix / } ) {
    my $index = $text.comb( :skip-empty ).first( * ~~ $prefix, :k );
    ( $text.comb[ 0 .. $index ].join.flip ~ $text.comb[ $index + 1 .. * - 1 ].join ).say;
}

```
<br/>
<br/>

I first search for the first index of the given char into the string, and then contanate the slices flipping the leftmost.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 341 - Task 1 - PL/Perl Implementation

Same approach as in Raku, with a nested loop.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc341.task1_plperl( text, text[] )
RETURNS SETOF text
AS $CODE$

   my ( $text, $keys ) = @_;

   for my $word ( split /\s+/, $text ) {
       my $found = 0;
       for my $key ( $keys->@* ) {
       	   $found += scalar grep { $_ eq $key } split( //, $word );
	   last if ( $found );
       }

       return_next( $word ) unless ( $found );
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 341 - Task 2 - PL/Perl Implementation

Similar to the Raku approach, use a slicing and reverse the leftmost.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc341.task2_plperl( text, text )
RETURNS text
AS $CODE$

   my ( $text, $prefix ) = @_;

   my $index = index( $text, $prefix );
   return $index unless( $index );

   my @chars = split //, $text;
   return join( '',
                reverse( @chars[ 0 .. $index ] ),
		@chars[ $index + 1 .. $#chars ] );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 341 - Task 1 - PL/PgSQL Implementation

Same approach as in the previous implementations, does a nested loop to get the words and check every letter to remove them from the list.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc341.task1_plpgsql( sentence text, keys text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	current_word text;
	ko int;
	k text;
BEGIN
	FOR current_word IN SELECT w FROM regexp_split_to_table( sentence, '\s+' ) w LOOP
	    ko := 0;

	    FOREACH k IN ARRAY keys LOOP
	    	IF current_word ~ k THEN
		   ko := ko + 1;
		END IF;
	    END LOOP;

	    IF ko = 0 THEN
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
## PWC 341 - Task 2 - PL/PgSQL Implementation

A quite verbose approach to create two arrays with the parts of the string, and a `while` loop to reverse the leftmost.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc341.task2_plpgsql( word text, p text )
RETURNS text
AS $CODE$
DECLARE
	i int;
	j int;
	tt text[];
	r text := '';
BEGIN
	i := position( p IN word );
	tt := regexp_split_to_array( word, '' );

	j := i;
	WHILE i > 0 LOOP
	      r := r || tt[ i ];
	      i := i - 1;
	END LOOP;

	r := r || array_to_string( tt[ j + 1 : ], '' );

	RETURN r;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 341 - Task 1 - PostgreSQL PL/Java Implementation

Similar approach as in other implementations, but this time I remove the wrong strings from the original list of words.


<br/>
<br/>
```java
    public static final String[] task1_pljava( String sentence, String[] keys ) throws SQLException {
	   logger.log( Level.INFO, "Entering pwc341.task1_pljava" );

	   List<String> words = new LinkedList<String>();
	   for ( String w : sentence.split( "\\s+" ) )
	       words.add( w );

	   Iterator<String> iter = words.iterator();
	   while ( iter.hasNext() ) {
	       String current_word = iter.next();
	       boolean ok = true;
	       for ( String k : keys )
	       	if ( current_word.contains( k ) )
	   	      ok = false;

	       if ( ! ok )
	         iter.remove();
	   }

	   String result[] = new String[ words.size() ];
	   int i = 0;
	   for ( String w : words )
	       result[ i++ ] = w;

	   return result;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 341 - Task 2 - PostgreSQL PL/Java Implementation

In this case, I use `StringBuilder` to perform the `reverse()` of the leftmost string quite quickly.


<br/>
<br/>
```java
    public static final String task2_pljava( String word, String prefix ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc341.task2_pljava" );

		int index = word.indexOf( prefix, 1 );
		StringBuilder builder = new StringBuilder();

		builder.append( word.substring( 0, index ) );
		builder.reverse();
		return builder.toString() + word.substring( index + 1 );

    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 341 - Task 1 - Python Implementation

Similar implementation to Raku.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    sentence = args[ 0 ]
    keys     = args[ 1 : ]
    result   = []

    for word in sentence.split():
        found = False

        for k in keys:
            if k in word:
                found = True

        if not found:
            result.append( word )

    return ' '.join( result )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 341 - Task 2 - Python Implementation

Similar to the PL/Perl approach.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    word   = args[ 0 ]
    prefix = args[ 1 ]

    index = word.index( prefix )

    return word[ index : 0 : -1 ] + word[ index : ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
