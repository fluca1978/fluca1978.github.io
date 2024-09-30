---
layout: post
title:  "Perl Weekly Challenge 289: Arrays Everywhere!"
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

# Perl Weekly Challenge 289: Arrays Everywhere!

This post presents my solutions to the [Perl Weekly Challenge 289](https://perlweeklychallenge.org/blog/perl-weekly-challenge-289/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 289 - Task 1 - Raku](#task1)
- [PWC 289 - Task 2 - Raku](#task2)
- [PWC 289 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 289 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 289 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 289 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 289 - Task 1 in PL/Java](#task1pljava)
- [PWC 289 - Task 2 in PL/Java](#task2pljava)
- [PWC 289 - Task 1 in Python](#task1python)
- [PWC 289 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 289 - Task 1 - Raku Implementation

The first task was to find out the *third* max value in an array of integers. In the case the value was not there (i.e., the array was shorter), return the absolute max value.

<br/>
<br/>
```raku
sub MAIN( *@numbers is copy where { @numbers.grep( * ~~ Int ).elems == @numbers.elems } ) {
    @numbers = @numbers.sort.unique;
    @numbers.elems >= 3 ?? @numbers[ * - 3 ].say !! @numbers[ * - 1 ].say;
}

```
<br/>
<br/>

The trick is to sort the array and return the third element if present or the very first one.


<a name="task2"></a>
## PWC 289 - Task 2 - Raku Implementation

The second task was to scramble words passed as an array, where the first and last letter were to be kept the same and the middle letters to be scrambled. In the case of punctuation, that must remain in the same position.

<br/>
<br/>
```raku
sub MAIN( *@words ) {
    my @output;
    for @words -> $current_word  {
		my @letters = $current_word.comb( :skip-empty );
		my ( $index_first, $index_last ) = 0, @letters.elems - 1;
		while ( @letters[ $index_last ] !~~ /<[a .. z A .. Z 0 .. 9 ]>/ ) {
		    $index_last--;
		}

		my $new_word = ( @letters[ $index_first ],
				 @letters[ $index_first + 1 .. $index_last - 1 ].pick( $index_last - $index_first ).join( '' ),
				 @letters[ $index_last ],
				 @letters[ $index_last + 1 .. * - 1 ] ).join( '' );

		@output.push: $new_word;
    }

    @output.join( ' ' ).say;
}

```
<br/>
<br/>

The implementation is simple: I iterate over all the words, one at a time, and extract the very first letter. Then I get the last one, and in the case is punctuation, I keep going backward. Then I use `pick` on the slice of the array in the middle and rebuild the word.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 289 - Task 1 - PL/Perl Implementation


Same implementation as in Raku, but use an hash to sort the unique numbers.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc289.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $numbers ) = @_;

   my $classify = {};
   $classify->{ $_ }++ for ( $numbers->@* );

   my @sorted = sort keys $classify->%*;
   return $sorted[ -1 ] if ( @sorted < 3 );
   return $sorted[ -3 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 289 - Task 2 - PL/Perl Implementation

Use `List::Util::shuffle` to shuffle the mid array of a word, similarly to what done in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc289.task2_plperl( text[] )
RETURNS SETOF text
AS $CODE$

   use List::Util qw/shuffle/;
   my ( $words ) = @_;

   for my $current_word ( $words->@* ) {
       my @letters = split //, $current_word;
       my @new_word;

       my ( $first_index, $last_index ) = ( 0, $#letters );
       my ( $first, $last ) = ( @letters[ $first_index ], @letters[ $last_index ] );

       while ( $last !~ /[a-z0-9]/i ) {
       	     $last_index--;
	     $last = $letters[ $last_index ];
       }


       # shuffle the remaining part
       my @shuffled = shuffle @letters[ $first_index + 1 .. $last_index - 1 ];

       return_next( join '', $first, @shuffled, $last, @letters[ $last_index + 1 .. $#letters ] );
   }

return undef;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 289 - Task 1 - PL/PgSQL Implementation

Almost solved with a query.
The problem is that I need an `IF` to test if the array has enough elements to decide what to return.

<br/>
<br/>
```sql

CREATE OR REPLACE FUNCTION
pwc289.task1_plpgsql( n int[] )
RETURNS int
AS $CODE$
DECLARE
	found_max int;
BEGIN
	IF array_length( n, 1 ) > 3 THEN
		WITH s AS (
			SELECT v, row_number()  OVER ( ORDER BY v DESC ) AS r
			FROM unnest( n ) v
			GROUP BY v
		        ORDER BY v DESC
		)
		SELECT v
		INTO found_max
		FROM s
		WHERE
		r = 3
		;
	ELSE
		SELECT MAX( v )
		INTO found_max
		FROM unnest( n ) v;
	END IF;

	RETURN found_max;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 289 - Task 2 - PL/PgSQL Implementation

I use `string_agg` and `ORDER BY random()` to sort the slice of the array in a random way.
Also `substring` to split the pieces of the word to analyze.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc289.task2_plpgsql( words text[])
RETURNS SETOF text
AS $CODE$
DECLARE

	current_word text;
	first_index int;
	last_index int;
	new_word text;
BEGIN
	FOREACH current_word IN ARRAY words LOOP
		first_index := 1;
		last_index  := length( current_word );
		new_word := substring( current_word from first_index for 1 );

		WHILE NOT substring( current_word from last_index for 1 ) ~ '[a-z0-9A-Z]' LOOP
		      last_index := last_index - 1;
		END LOOP;


		WITH l AS ( SELECT v::text
			    FROM  regexp_split_to_table( substring( current_word from first_index + 1 for last_index - first_index - 1 ), '' ) v
   			    ORDER BY random() )
		SELECT string_agg( l.v, '' )
		INTO new_word
		FROM  l;


		RETURN NEXT substring( current_word from first_index for 1 )
		|| new_word
		|| substring( current_word from last_index  );
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
## PWC 289 - Task 1 - PostgreSQL PL/Java Implementation

I use the Stream API to produce a `sorted` array, and then extract the third or first value.

<br/>
<br/>
```java
    public static final int task1_pljava( int[] nums ) throws SQLException {
		List<Integer> sorted = IntStream.of( nums ).boxed().sorted( Collections.reverseOrder() )
		    .collect( Collectors.toList() );


		if ( sorted.size() >= 3 )
		    return sorted.get( 3 );
		else
		    return sorted.get( 0 );

    }

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 289 - Task 2 - PostgreSQL PL/Java Implementation

Same implementation as in Raku, but a lot more verbose.


<br/>
<br/>
```java
    public static final String[] task2_pljava( String[] words ) throws SQLException {
		List<String> result = new LinkedList<String>();

		for ( String current : words ) {
		    String[] letters = current.split( "" );


		    int last_index = letters.length - 1;
		    int first_index = 0;

		    Pattern pattern = Pattern.compile( "^[a-z0-9A-Z]$" );


		    while ( ! pattern.matcher( letters[ last_index ] ).find() )
				last_index--;

		    List<String> shuffling = new LinkedList<String>();
		    for ( int i = 1; i < last_index; i++ )
				shuffling.add( letters[ i ] );

		    Collections.shuffle( shuffling );

		    String current_result = letters[ first_index ] + String.join( "", shuffling );
		    for ( int i = last_index; i < letters.length; i++ )
				current_result += letters[ i ];

		    result.add( current_result );
		}

		String[] output = new String[ result.size() ];
		for ( int i = 0; i < output.length; i++ )
		    output[ i ] = result.get( i );

		return output;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 289 - Task 1 - Python Implementation

Same approach as in PL/Perl.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    sorted_nums = list( sorted( map( int, args ) ) )
    sorted_nums = sorted_nums[ ::-1 ]

    if len( sorted_nums ) >= 3:
        return sorted_nums[ 2 ]
    else:
        return sorted_nums[ 0 ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 289 - Task 2 - Python Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```python
import sys
import random

# task implementation
# the return value will be printed
def task_2( words ):
    new_words = []

    for word in words:
        current_word = []
        current_word.append( word[ 0 ] )

        last_index = len( word ) - 1
        while not str.isalpha( word[ last_index ] ) and not str.isdigit( word[ last_index ] ):
            last_index -= 1

        shuffling = list( map( str, word[ 1 : last_index ] ) )
        random.shuffle( shuffling )
        for r in shuffling:
            current_word.append( r )

        current_word.append( word[ last_index : ] )
        new_words.append( ''.join( current_word ) )

    return new_words



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
