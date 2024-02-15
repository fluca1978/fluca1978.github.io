---
layout: post
title:  "Perl Weekly Challenge 256: Valentine's Challenge"
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

# Perl Weekly Challenge 256: Valentine's Challenge

This post presents my solutions to the [Perl Weekly Challenge 256](https://perlweeklychallenge.org/blog/perl-weekly-challenge-256/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 256 - Task 1 - Raku](#task1)
- [PWC 256 - Task 2 - Raku](#task2)
- [PWC 256 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 256 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 256 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 256 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 256 - Task 1 in PL/Java](#task1pljava)
- [PWC 256 - Task 2 in PL/Java](#task2pljava)
- [PWC 256 - Task 1 in Python](#task1python)
- [PWC 256 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 256 - Task 1 - Raku Implementation

The first task was about finding out how many times a word appears reversed in a list of given words, and counting the max pair of word and reversred words in the list.

<br/>
<br/>
```raku
sub MAIN( *@words ) {
    my %counting;
    for @words -> $current-word {
		%counting{ $current-word } = @words.grep( { $_ ~~ $current-word.flip } ).elems;
    }

    %counting.values.max.say;
}

```
<br/>
<br/>

I use a `%couting` hash to classify every word with the counting of the matches with the reversed (i.e., `flip`) word. Then I print out the `max` value find in the hash.


<a name="task2"></a>
## PWC 256 - Task 2 - Raku Implementation

This task was about taking two words as input, and print out a letter from every word mixing the content.

<br/>
<br/>
```raku
sub MAIN( Str $left, Str $right ) {
    my $output = ( $left.comb Z $right.comb ).flat.join;

    if ( $left.chars > $right.chars ) {
		$output ~= $left.comb[ $right.chars .. * ].join;
    }
    elsif ( $left.chars < $right.chars ) {
		$output ~= $right.comb[ $left.chars .. * ].join;
    }

    $output.say;
}

```
<br/>
<br/>

There is a name for mixing the content of the words (or in general, of arrays) interleaving every element: `zip`. The problem of the `zip` function is that it ends as soon as one side of the mixing parts ends, while the task was requiring to use all the characters from every word. Therefore, after obtaining the `zip` result, I check if either the first or second word is longer than the other, and extract the remaining chars by *slicing* the `comb` and rejoining into the `$output` string.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 256 - Task 1 - PL/Perl Implementation

The implementation is the same as in Raku: I use a `$coutnging` hash ref to store each word and the copunt of the reversed word in the list, and then iterare over the values to get the max counting.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc256.task1_plperl( text[] )
RETURNS int
AS $CODE$

   my ( $words ) = @_;

   my $counting = {};
   for my $current ( $words->@* ) {
     $counting->{ $current } = scalar grep( { $_ eq reverse( $current ) } $words->@* )	;
   }


   my $max = 0;
   for ( values $counting->%* ) {
       $max = $_ if ( $_ > $max );
   }

   return $max;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 256 - Task 2 - PL/Perl Implementation

Here I implement the `zip` behaviour within the function.
I first extract the array of every letter from the words, then I iterate with an `$index` that has to be less than the number of chars in the first or second word, so that the `while` loop continues even if one word has exhausted its letters.
Within the `while` loop, I add every character pointed by `$index` if the word has other characters remaining.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc256.task2_plperl( text, text )
RETURNS text
AS $CODE$

   my ( $left, $right ) = @_;
   my $output = '';

   my @left_chars = split //, $left;
   my @right_chars = split //, $right;

   my $index = 0;
   while ( $index < @left_chars || $index < @right_chars ) {
   	 $output .= $left_chars[ $index ] if ( $index < @left_chars );
	 $output .= $right_chars[ $index] if ( $index < @right_chars );
	 $index++;
   }

   return $output;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Another possible solution could have been to use the array of letter as *stacks*:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc256.task2_plperl( text, text )
RETURNS text
AS $CODE$

   my ( $left, $right ) = @_;
   my $output = '';

   my @left_chars = split //, $left;
   my @right_chars = split //, $right;

   while ( @left_chars @right_chars ) {
   	 $output .= shift( @left_chars ) if ( @left_chars );
	 $output .= shift( @right_chars ) if ( @right_chars):
	 $index++;
   }

   return $output;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 256 - Task 1 - PL/PgSQL Implementation

Here the task can be solved with a single nested query.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc256.task1_plpgsql( words text[] )
RETURNS text
AS $CODE$
	SELECT max( o )
	FROM ( SELECT w, count(*) as o
	       FROM unnest( words ) w, unnest( words ) ww
	       WHERE w = reverse ( ww )
	       GROUP BY w
	      );

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The outer query extract the `max` value of `o`, which in turn is the `count` of the number of rows where a word `w` matches the reverse of all the other words.


<a name="task2plpgsql"></a>
## PWC 256 - Task 2 - PL/PgSQL Implementation

I use a temporary table to store the characters coming from the left and right words, as well as the ordering of the letter within the word.
Then I extract the max value of the ordering, that is the max number of characters against the two words, and then iterate from `1` to the max value. At every iteration I extract a letter from `l` or `r` with the same ordering, and if they are not null, I append the characters to the `output` string. At the end, the string is returned.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc256.task2_plpgsql( lw text, rw text )
RETURNS text
AS $CODE$
DECLARE
	max_index int;
	output text := '';
	next_char char;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS words( l char, r char, o int );
	TRUNCATE words;

	INSERT INTO words( l, o )
	SELECT v, row_number() over ()
	FROM regexp_split_to_table( lw, '' ) v;

	INSERT INTO words( r, o )
	SELECT v, row_number() over ()
	FROM regexp_split_to_table( rw, '' ) v;

	SELECT max( o )
	INTO max_index
	FROM words;

	FOR i IN 1 .. max_index LOOP
	    SELECT l
	    INTO next_char
	    FROM words
	    WHERE o = i
	    AND l IS NOT NULL;

	    IF FOUND AND next_char IS NOT NULL THEN
	       output := output || next_char;
	    END IF;

	    SELECT r
	    INTO next_char
	    FROM words
	    WHERE o = i
	    AND r IS NOT NULL;

	    IF FOUND AND next_char IS NOT NULL THEN
	       output := output || next_char;
	    END IF;
	END LOOP;

	RETURN output;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 256 - Task 1 - PostgreSQL PL/Java Implementation

Here I implement an utility method `count` that taks a word and the list of words as input.
The `count` method reverses the input word and iterate on every word in the list to see if it does match, counting how many matches are found.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "256",
	      onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( String[] words ) throws SQLException {
		logger.log( Level.INFO, "Entering task1_pljava" );

		int max = 0;
		for ( String current : words ) {
		    int count = count( current, words );
		    if ( count > max )
				max = count;
	   }

	   return max;

    }

    private static final int count( String word, String[] words ) {
		int count = 0;
		StringBuilder builder = new StringBuilder();
		builder.append( word );
		word = builder.reverse().toString();

		for ( String w : words )
		    if ( w.equals( word ) )
				count++;

		return count;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 256 - Task 2 - PostgreSQL PL/Java Implementation

The implementation is quite simple: I build a `StringBuffer` and compute the `max_index` that is the max length of the two words.
Then I iterate and insert a character from each word depending if the word as still characters. At the end, I return the string buffer converted to a string.

<br/>
<br/>
```java
public class Task2 {
    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "256",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task2_pljava( String left, String right ) throws SQLException {
		logger.log( Level.INFO, "Entering task2_pljava" );

		StringBuffer buffer = new StringBuffer( left.length() + right.length() );

		int max_index = Math.max( left.length(), right.length() );
		for ( int index = 0; index < max_index; index++ ) {
		    if ( index < left.length() )
				buffer.append( left.charAt( index ) );
		    if ( index < right.length() )
				buffer.append( right.charAt( index ) );
		}

		return buffer.toString();
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 256 - Task 1 - Python Implementation

The implementation is similar to PL/Java one: I iterate on every word in the list and `filter` to match the reversed word, and then count the `len` of the resulting list. The variable `max` is used to keep track of the highest value found.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    max = 0
    for w in args:
        count = len( list( filter( lambda rw: rw == ''.join( reversed( w ) ), args ) ) )
        if count > max:
            max = count

    return max


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 256 - Task 2 - Python Implementation

The implementation is very similar to the PL/Java one: I compute the `max_index` representing the max length of the words, and then iterata appending a character to the `output` string.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    left   = args[ 0 ]
    right  = args[ 1 ]
    output = ''

    max_index = max( len( left ), len( right ) )

    for i in range( 0, max_index ):
        if i < len( left ):
            output += left[ i ]
        if i < len( right ):
            output += right[ i ]

    return output


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
