---
layout: post
title:  "Perl Weekly Challenge 278: 200 CHALLENGES (almost) IN A ROW!"
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

# Perl Weekly Challenge 278: 200 CHALLENGES (almost) IN A ROW!

This post presents my solutions to the [Perl Weekly Challenge 278](https://perlweeklychallenge.org/blog/perl-weekly-challenge-278/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 278 - Task 1 - Raku](#task1)
- [PWC 278 - Task 2 - Raku](#task2)
- [PWC 278 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 278 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 278 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 278 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 278 - Task 1 in PL/Java](#task1pljava)
- [PWC 278 - Task 2 in PL/Java](#task2pljava)
- [PWC 278 - Task 1 in Python](#task1python)
- [PWC 278 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 278 - Task 1 - Raku Implementation

The first task was to sort out words in a list where the last character of each word is the ordering value.

<br/>
<br/>
```raku
sub MAIN( *@words ) {
    my %phrase;
    for @words {
		/ ^ (<[a..zA..Z]>+) (\d) $ /;
		%phrase{ $/[ 1 ] } = $/[ 0 ].Str;
    }

    %phrase.sort.map( { .value } ).join( ' ' ).say;
}

```
<br/>
<br/>


I use a `%phrase` hash to sort the words with their ordinality as a key. Therefore, it does suffice to `sort` the hash, then to `map` to the hash values and join with a space.



<a name="task2"></a>
## PWC 278 - Task 2 - Raku Implementation

The second task was about sorting the letters in a word up to a given letter, leaving the rightmost part of the word unchanged.

<br/>
<br/>
```raku
sub MAIN( Str $word, Str $char ) {
    $word.say and exit if ( $word !~~ / $char / );

    my @chars;
    for $word.comb {
		@chars.push: $_;
		last if $_ eq $char;
    }

    say @chars.sort.join( '' ) ~ $word.comb[ @chars.elems .. * ].join( '' );
}

```
<br/>
<br/>

The idea is to set an array `@chars` that will store every character up to the given letter. Then I join two strings, where the leftmost is the sorted `@chars` array and the rightmost is the remaining part of the original string.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 278 - Task 1 - PL/Perl Implementation

The idea is the same as in Raku: I use an hash `%sorted` keyed by the number at the end of the word and with the value as the word.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc278.task1_plperl( text[] )
RETURNS SETOF text
AS $CODE$

   my ( $words ) = @_;
   my %sorted;

   for ( $words->@* ) {
       my ( $word, $position ) = ( $_ =~ / ^ ([a-zA-Z]+) (\d+) $ /x );
       $sorted{ $position } = $word;
   }

   return_next( $sorted{ $_ } ) for ( sort keys %sorted );
   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 278 - Task 2 - PL/Perl Implementation

Same implementation as in Raku: I push all characters up to the given `$char` into a `@left` array, that is then sorted and joined with the remaining rightmost part of the original word.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc278.task2_plperl( text, text )
RETURNS text
AS $CODE$

   my ( $word, $char ) = @_;

   return $word if ( $word !~ / $char /x );

   my @original = split //, $word;
   my @chars;
   for ( @original ) {
       push @chars, $_;
       last if $_ eq $char;
   }

   return join( '', sort( @chars ),  @original[ $#chars + 1 .. $#original ] );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 278 - Task 1 - PL/PgSQL Implementation

A single query does suffice.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc278.task1_plpgsql( words text[] )
RETURNS SETOF text
AS $CODE$

   WITH numbered AS (
      SELECT substring( v::text from '^[a-zA-Z]+' ) as word
           , substring( v::text from '\d+$' )::int as index
      FROM unnest( words ) v
   )
   SELECT word
   FROM numbered
   ORDER BY index
   ;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


The `numbered` materialization is made by two columns, one with the `word` part and the other with the `index` numbering, both extracted with a regular expression. Then there is the selection of every word ordering by the index.


<a name="task2plpgsql"></a>
## PWC 278 - Task 2 - PL/PgSQL Implementation

Again, a single query does suffice.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc278.task2_plpgsql( w text, c char )
RETURNS text
AS $CODE$

   WITH letters( l, i ) AS (
   	SELECT *
	FROM regexp_split_to_table( w, '' )
	WITH ORDINALITY AS v( text, int )
   )
   , first_part AS (
     SELECT l
     FROM letters
     WHERE i <= position( c IN w )
     ORDER BY l
   )
   , second_part AS (

     SELECT l
     FROM letters
     WHERE i > position( c IN w )
     ORDER BY i
   )
   , all_letters AS (
   SELECT l
   FROM first_part
   UNION ALL
   SELECT l
   FROM second_part
   )
   SELECT string_agg( l, '' )
   FROM all_letters
   ;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>


I materialize two tables, `first_part` with all the letters before the index position of the given letter, and the `second_part` with the remaining. Then, I build on top the above the `all_letters` the `UNION ALL` of both queries. Last, I select the `string_agg` which is the join of the letters from both the parts.


# Java Implementations

<a name="task1pljava"></a>
## PWC 278 - Task 1 - PostgreSQL PL/Java Implementation

I use the Stream API to build the same `sorted` hash map as in the other implementations.

<br/>
<br/>
```java
    public static final String task1_pljava( String[] words ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc278.task1_pljava" );

		final Map<Integer, String> sorted = new HashMap<Integer, String>();
		Stream.of( words ).forEach( word -> {
			Pattern pattern  = Pattern.compile( "^([a-zA-Z]+)(\\d+)+$" );
			Matcher m = pattern.matcher( word );
			if ( m.matches() ) {
			    sorted.put( Integer.parseInt( m.group( 2 ) ),
					m.group( 1 ) );
			}
		    } );

		return sorted.entrySet().stream()
		    .sorted( Comparator.comparing( Map.Entry::getKey ) )
		    .map( Map.Entry::getValue )
		    .collect( Collectors.joining( " " ) );
    }
```
<br/>
<br/>

Note that in Java, the grouping with regular expressions starts from `1`.
Then I `collect` the values of the hashmap after having sorted them on keys.


<a name="task2pljava"></a>
## PWC 278 - Task 2 - PostgreSQL PL/Java Implementation

Again, I use the Steam API to solve the problem.

<br/>
<br/>
```java
    public static final String task2_pljava( String word, String c ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc278.task2_pljava" );

		if ( ! word.contains( c ) )
		    return word;

		final List<String> left = new LinkedList<String>();
		final List<String> right = new LinkedList<String>();
		final boolean[] found = { false };
		Stream.of( word.split( "" ) ).forEach( letter -> {
			if ( found[ 0 ] )
			    right.add( letter );
			else {
			    left.add( letter );

			    if ( letter.equals( c ) )
				found[ 0 ] = true;
			}
		    } );

		Collections.sort( left );
		return left.stream().collect( Collectors.joining() ) + right.stream().collect( Collectors.joining() );
    }
```
<br/>
<br/>

I build two strings lists `left` and `right` to append all the characters for both the parts. I then `forEach` on the list of characters and if yet not found the needle, append the letter to the `left` list, otherwise to the `right` list. Then I sort only the `left` list and append the two `collect`ion of strings.

# Python Implementations

<a name="task1python"></a>
## PWC 278 - Task 1 - Python Implementation

Same implementation of the above solutions.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    sorted_words = {}
    pattern      = re.compile( "^([a-zA-Z]+)(\\d)+$" )

    for w in args:
        m = pattern.match( w )
        sorted_words[ str( m.group( 2 ) ) ] = m.group( 1 )

    sorted_words = dict( sorted( sorted_words.items() ) )
    return ' '.join( sorted_words.values() )



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>

I use a dictionary, keyed by the numbering of each word. With `sorted` I then sort the dictionary and join all the values.


<a name="task2python"></a>
## PWC 278 - Task 2 - Python Implementation

Similar approach as in Raku: I append every character up to the needle into a `left` list.
last, I join the sorted list with the remainder of the word.


<br/>
<br/>
```python
 import sys

# task implementation
# the return value will be printed
def task_2( args ):
    word = args[ 0 ]
    c    = args[ 1 ]

    if not c in word:
        return word

    left = []

    for x in word:
        left.append( x )
        if x == c:
            break

    return ''.join( list( sorted( left ) ) ) + word[ len( left )  : len( word )  ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
