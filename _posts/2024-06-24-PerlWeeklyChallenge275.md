---
layout: post
title:  "Perl Weekly Challenge 275: pipelines!"
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

# Perl Weekly Challenge 275: pipelines!

This post presents my solutions to the [Perl Weekly Challenge 275](https://perlweeklychallenge.org/blog/perl-weekly-challenge-275/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 275 - Task 1 - Raku](#task1)
- [PWC 275 - Task 2 - Raku](#task2)
- [PWC 275 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 275 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 275 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 275 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 275 - Task 1 in PL/Java](#task1pljava)
- [PWC 275 - Task 2 in PL/Java](#task2pljava)
- [PWC 275 - Task 1 in Python](#task1python)
- [PWC 275 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 275 - Task 1 - Raku Implementation

The first task was about finding out, given a sentence, how many words can be written without using a given list of *bad characters*.

<br/>
<br/>
```raku
sub MAIN( Str $sentence, *@keys ) {
    my @ok-words;

    for $sentence.lc.split( / \s+ / ) -> $word {
		my $ok = True;

		for @keys -> $needle {
		    $ok = False if ( $word ~~ / $needle / );
		    last if ! $ok;
		}

		@ok-words.push: $word if $ok;
    }

    @ok-words.elems.say;
}

```
<br/>
<br/>

The idea is simple and I use a nested loop: first I split the `$sentence` into `$word`s and then checks every single `@keys` to see if it is contained into the word. If it is not, I append the word (as it is good) into `@ok-words` so that I can count and print them out.


<a name="task2"></a>
## PWC 275 - Task 2 - Raku Implementation

Given an aplhanumeric string, substitute every digit with the previous character moved forward of the value of the digit itself.

<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / ^ <[a..zA..Z]> <[a..zA..Z0..9]>+ $ / } ) {
    my $previous;
    my @result;

    my $index = 0;
    my %alphabet;
    %alphabet{ $_ } = $index++  for 'a' .. 'z';



    $string.comb.map( -> $current is copy {
			    if ( $current.lc ~~ / <[a..z]> / ) {
					# it is a letter
					$previous = $current;
			    }
			    else {
					# it is a number
					$current = %alphabet.pairs.grep( { $_.value == ( %alphabet{ $previous } + $current.Int ) } )[ 0 ].key;
			    }

			    @result.push: $current;
			} );


    @result.join.say;
}

```
<br/>
<br/>

I first build an `%alphabet` to store all the letters and their indexes.
Then I `map` every letter so that if it is a letter I simply append to the `@result` array, otherwise I extract the lette5r and its index from the `%alphabet` and sum the integer value of the `$current` letter and get the resulting letter.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 275 - Task 1 - PL/Perl Implementation

The implementation is essentially the same as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc275.task1_plperl( text, text[] )
RETURNS int
AS $CODE$

   my ( $string, $keys ) = @_;
   my @ok_words;

   for my $word ( split /\s+/, lc $string ) {
       my $ok = 1;
       for my $wrong_key ( $keys->@* ) {
       	   $ok = 0 if ( $word =~ /$wrong_key/ );
	   last if ( ! $ok );
       }

       push @ok_words, $word if ( $ok );
   }

   return scalar( @ok_words );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 275 - Task 2 - PL/Perl Implementation

I found this a little simpler to implement since Perl, at least to my memory, provides simpler ways to manipulate ASCII chars.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc275.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;
   my @alphabet = 'a' .. 'z';
   $string = lc $string;
   my @result;
   my $previous;

   for my $letter ( split //, $string ) {
       if ( $letter =~ /[a-z]/ ) {
       	  $previous = $letter;
       }
       else {
       	  $letter = chr( ord( 'a' ) + int( $letter ) );
       }

       push @result, $letter;
   }

   return join( '', @result );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 275 - Task 1 - PL/PgSQL Implementation

The idea is the same as in PL/Perl:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc275.task1_plpgsql( s text, k text[] )
RETURNS int
AS $CODE$
DECLARE
	ok boolean;
	current_k text;
	current text;
	word_counter int := 0;
BEGIN

	FOR current IN SELECT word FROM regexp_split_to_table( s, '\s+' ) word LOOP
    	    ok := true;

	    FOREACH current_k IN ARRAY k LOOP
	    	    IF current ~ current_k THEN
		       ok := false;
		    END IF;
	    END LOOP;

	    IF ok THEN
	       word_counter := word_counter + 1;
	    END IF;
	END LOOP;

	RETURN word_counter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


I use the `~` operator that performs a regular expression match to check if the `urrent` word contains the `current_k` bad letter.
Unlike the Perl implementation, here I count only the words, without accumulating them.



<a name="task2plpgsql"></a>
## PWC 275 - Task 2 - PL/PgSQL Implementation

Same approach as before: I split the word into all its letter, check if a letter is a number or a letter and proceed accumulating the result into the `output` string.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc275.task2_plpgsql( s text )
RETURNS text
AS $CODE$
DECLARE
	output text := '';
	previous text;
	c text;
BEGIN

	FOR c IN SELECT v FROM regexp_split_to_table( s, '' ) v LOOP
	    IF c ~ '[a-z]' THEN
	       previous := c;
	    ELSE
		c := chr( c::int + ascii( 'a' ) );
	    END IF;

	    output := output || c;
	END LOOP;

	return output;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 275 - Task 1 - PostgreSQL PL/Java Implementation

I implemented this using the Stream API.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc275",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( String sentence, String[] keys ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc275.task1_pljava" );

		List<Pattern> patterns = Stream.of( keys )
		    .map( letter -> { return Pattern.compile( String.format( "%s", letter ), Pattern.CASE_INSENSITIVE ); } )
		    .collect( Collectors.toList() );

		return (int) Stream.of( sentence.split( "\\s+" ) )
		    .filter( word -> {
			    for ( Pattern p : patterns )
					if ( p.matcher( word ).find() )
					    return false;

			    return true;
			} ).count();

    }
}

```
<br/>
<br/>


First of all, I implement a list of `Pattern` objects, where every object is a pattern matching a single letter out of the `keys` array.
Then I `split` the sentence into words, and `filter` against every pattern to see if it matches. If it does not match, the word is good, so must be kept, otherwise must be discarded. At the then, I `count()` the number of words kept.


<a name="task2pljava"></a>
## PWC 275 - Task 2 - PostgreSQL PL/Java Implementation

Implemented with the Stream API.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc275",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task2_pljava( String text ) throws SQLException {
	logger.log( Level.INFO, "Entering pwc275.task2_pljava" );

	Pattern digit = Pattern.compile( "\\d", Pattern.CASE_INSENSITIVE );
		final String[] previous = new String[ 1 ];

		return Stream.of( text.split( "" ) )
		    .map( current -> {
			    if ( digit.matcher( current ).find() ) {
				// a digit

			        current = String.format( "%c", ( (int) previous[ 0 ].charAt( 0 ) + Integer.parseInt( current ) ) );

			    }
			    else {
					previous[ 0 ] = current;
			    }

			    return current;
			} )
		    .collect( Collectors.joining("") );
    }
}

```
<br/>
<br/>


I build a `Pattern` that matches only digits, and `split` the word into every character, remapping every char into its alphabetic form or a trasnlated (i.e., shifted by index) one. At the end, I `collect` by means of `joining`.


# Python Implementations

<a name="task1python"></a>
## PWC 275 - Task 1 - Python Implementation

Quite simple implementation, counting only the words without accumulating them.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    sentence = args[ 0 ]
    keys     = args[ 1: ]
    words_ok = 0

    for word in sentence.lower().split():
        bad = 0
        for k in keys:
            k = k.lower()
            if k in word:
                bad += 1
                break

        if bad == 0:
            words_ok += 1

    return words_ok

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 275 - Task 2 - Python Implementation

Straighforward implementation.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    string = args[ 0 ]
    previous = None
    result   = ""

    for c in string:
        if c.isdigit():
            c = chr( int( c ) + 97 )
        else:
            previous = c

        result += c

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
