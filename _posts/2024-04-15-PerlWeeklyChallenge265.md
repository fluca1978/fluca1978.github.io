---
layout: post
title:  "Perl Weekly Challenge 265: arrays and dictionaries"
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

# Perl Weekly Challenge 265: arrays and dictionaries

This post presents my solutions to the [Perl Weekly Challenge 265](https://perlweeklychallenge.org/blog/perl-weekly-challenge-265/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 265 - Task 1 - Raku](#task1)
- [PWC 265 - Task 2 - Raku](#task2)
- [PWC 265 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 265 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 265 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 265 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 265 - Task 1 in PL/Java](#task1pljava)
- [PWC 265 - Task 2 in PL/Java](#task2pljava)
- [PWC 265 - Task 1 in Python](#task1python)
- [PWC 265 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 265 - Task 1 - Raku Implementation

The first task was to find out if, in a given array of numbers, there is at least one that appears for more than the `33%`. In the case there are more than one, the smallest one must be shown.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my %pct;
    %pct{ $_ } += 1 / @nums.elems for @nums;

    my @good_ones = %pct.kv.grep( -> $k, $v { $v >= 33 / 100 ?? True !! False } );
    exit if ! @good_ones;

    @good_ones.map( { $_[ 0 ] } ).sort.head.say;

}

```
<br/>
<br/>


The idea is simple: I use a `%pct` hash to store every number and the percentage it appears in the list. Then I place only those values that appear for more than `1/3` into the `@good_ones` array, that is then sorted and from which I extract only the first (i.e., smallest) element.

<a name="task2"></a>
## PWC 265 - Task 2 - Raku Implementation

The second task was about finding the smallest completion word from a template and a list of words.
The idea is that the template defines the occurrencies of letters (not spaces and numbers) that have to appear in any of the given word list.


<br/>
<br/>
```raku
sub MAIN( Str $needle, *@strings ) {

    my %letters;
    %letters{ $_.lc }++ if ( $_ !~~ / <[0..9]> | \s / ) for $needle.comb;

    my %completing-words;

    for @strings -> $current {
		my $ok = True;
		for %letters.kv -> $letter, $count {
		    $ok = False and last if $current.comb.grep( $letter ).elems < $count;
		}
		%completing-words{ $current.chars } = $current if $ok;
    }

    %completing-words{ %completing-words.keys.min }.say;
}
```
<br/>
<br/>


First, I extract only the lower case letters and their occurrencies, placing them into the `%letters` hash.
Then, I iterate over the `@strings` and for every string I check if all the letters match for at least the specified quantity, and in the case I append the word to the `%completing-words` hash. Such hash is keyed by the word length, so that only word can appear for a given length and that I can simply extract the shortes word.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 265 - Task 1 - PL/Perl Implementation

Same implementation as in Raku, a little more verbose.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc265.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;

   my $pct = {};
   $pct->{ $_ } += 1 / scalar( $nums->@* ) for ( $nums->@* );
   my @good_ones;

   for ( keys $pct->%* ) {
       next if $pct->{ $_ } < 33 / 100;
       push @good_ones, $_;
   }

   return undef if ( ! @good_ones );
   return ( sort @good_ones )[ -1 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 265 - Task 2 - PL/Perl Implementation

Same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc265.task2_plperl( text, text[] )
RETURNS text
AS $CODE$

   my ( $needle, $words ) = @_;

   my $letters = {};

   for ( split( //, $needle ) ) {
       next if /[0-9]|\s/;
       $letters->{ lc $_ }++;
   }


   my $found_words = {};

   for my $current ( $words->@* ) {
       my $ok = 1;
       for my $letter ( keys $letters->%* ) {
       	   $ok = 0 and last if ( $letters->{ $letter } > scalar( grep { $_ eq $letter } split( //, $current ) ) );
       }

       next if ( ! $ok );
       $found_words->{ length $current } = $current if ( $ok );
   }

   return $found_words->{ ( sort keys $found_words->%* )[ 0 ] };

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 265 - Task 1 - PL/PgSQL Implementation

A single query can do the trick!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc265.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$

	SELECT v
	FROM unnest( nums ) v
	GROUP BY v
	HAVING count( v ) / array_length( nums, 1 ) >= ( 33 / 100 )
	ORDER BY v ASC
	LIMIT 1;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

Thanks to the `GROUP BY` and `HAVING` clauses, every row in the table provides the number and its occurrencies. Note how `HAVING` is filtering only thos rows with a percentage greater or equal to `33%`.



<a name="task2plpgsql"></a>
## PWC 265 - Task 2 - PL/PgSQL Implementation

The approach is the same as in PL/Perl, but in order to use *hash*-like structures, I use two temporary tables.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc265.task2_plpgsql( needle text, words text[] )
RETURNS text
AS $CODE$
DECLARE
	current text;
	letter text;
	size   int;
	current_count int;
	ok bool;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS letters( l text, c int );
	TRUNCATE TABLE letters;

	CREATE TEMPORARY TABLE IF NOT EXISTS found_words( w text, c int );
	TRUNCATE TABLE found_words;

	INSERT INTO letters
	SELECT lower( v ), count( v )
	FROM unnest( regexp_split_to_array( needle, '' ) ) v
	WHERE v NOT IN ( '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', ' ' )
	GROUP BY lower( v );


	FOREACH current IN ARRAY words LOOP
		ok := true;

		FOR letter, size IN SELECT l, c FROM letters LOOP
		    SELECT count( v )
		    INTO current_count
		    FROM unnest( regexp_split_to_array( current, '' ) ) v
		    WHERE  v = letter;

		    IF NOT FOUND OR current_count < size THEN
		       ok := false;
		       EXIT;
		    END IF;
		END LOOP;

		IF ok THEN
		   INSERT INTO found_words
		   SELECT current, length( current );
		END IF;
	END LOOP;

	SELECT w
	INTO current
	FROM found_words
	ORDER BY c ASC
	LIMIT 1;

	RETURN current;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The `letters` table stores the letters and the counting of the pattern, while `found_words` keeps track of the words that satisfy the conditions.
Therefore, once the tables have been appropriately populated, it does suffice to extract the first word with the smallest length.

# Java Implementations

<a name="task1pljava"></a>
## PWC 265 - Task 1 - PostgreSQL PL/Java Implementation

A different approach here: I use the `pct` hash to store every digit with its counting.
I iterate over the given array, and update the `pct` hash, keeping also the `minFound` value updated to very lowest value that appears more than `33%`.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc265",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final Integer task1_pljava( int[] nums ) throws SQLException {

		Map<Integer, Double> pct = new HashMap<Integer, Double>();
		Integer minFound = null;
		for ( int current : nums ) {
		    double value = 1 / ( double ) nums.length;
		    if ( pct.containsKey( current ) )
				value += pct.get( current );

		    pct.put( current, value );

		    if ( value >= 0.33 )
				if ( minFound == null || current < minFound )
					minFound = current;
		}

		return minFound;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 265 - Task 2 - PostgreSQL PL/Java Implementation

A very verbose approach.


<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc265",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task2_pljava( String needle, String[] words ) throws SQLException {


		Map<Character, Integer> letters = new HashMap<Character, Integer>();
		List<String> found_words = new LinkedList<String>();

		// classify the letters
		for ( Character c : needle.toLowerCase().toCharArray() )
		    if ( Character.isLetter( c ) ) {
				int count = 1;
				if ( letters.containsKey( c ) )
					count += letters.get( c );

	           letters.put( c, count );
		    }

		for ( String current : words ) {
		    int ok = 0;
		    Map<Character, Integer> currentLetters = new HashMap<Character, Integer>();

		    for ( Character c : current.toLowerCase().toCharArray() ) {
				int value = 1;
				if ( currentLetters.containsKey( c ) )
					value += currentLetters.get( c );

	            currentLetters.put( c, value );
		    }

		    for ( Character c : letters.keySet() ) {
				if ( ! currentLetters.containsKey( c )
	                || currentLetters.get( c ) < letters.get( c ) ) {
					ok = 0;
					break;
				}
				else
					ok++;
		    }

		    logger.log( Level.INFO, current + " ===> " + ok );
		    if ( ok == letters.keySet().size() )
			found_words.add( current );
		}

		if ( found_words.isEmpty() )
		    return null;

	    Collections.sort( found_words, new Comparator<String>() {
			@Override
			public int compare( String a, String b ) {
				return a.length() - b.length();
			}
		} );
		return found_words.get( 0 );
    }
}

```
<br/>
<br/>

The idea is the same as in other implementations: I keep a `found_words` list of words that satisfy the conditions, and a `letters` hash that keeps the counting of the letters from the pattern.
Then I iterate over the words, split them into characters and place them into an appropriate hashmap, and count how many letters match the required from the pattern. If all conditions are met, return the smallest word.

# Python Implementations

<a name="task1python"></a>
## PWC 265 - Task 1 - Python Implementation

Same approach as in Raku: use `pct` as a dictionary that store the number and its occurrency percentage.
Then, iterate over the dictionary and search for the min value.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    pct = {}
    for x in args:
        if not x in pct:
            pct[ x ] = 0
        pct[ x ] += 1 / len( args )

    found = None
    for x in pct:
        if pct[ x ] >= ( 33 / 100 ) and ( found is None or int( x ) < found ):
            found = int( x )

    return found

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 265 - Task 2 - Python Implementation

Same approach as in the other implementations.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    args        = list( map( lambda x: x.lower(), args ) )
    needle      = args[ 0 ]
    words       = args[ 1: ]
    found_words = []

    letters = {}
    for l in needle:
        if l == ' ' or l.isdigit():
            continue

        if not l in letters:
            letters[ l ] = 0
        letters[ l ] += 1

    for current in words:
        ok = True
        for letter in letters:
            if letters[ l ] > len( list( filter( lambda x: x == l, current ) ) ):
                ok = False
                break
        if ok:
            found_words.append( current )

    found_words.sort(key = lambda x: len( x ) )
    return found_words[ 0 ]

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
