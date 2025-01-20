---
layout: post
title:  "Perl Weekly Challenge 306: Sorting and Nested Loops"
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

# Perl Weekly Challenge 306: Sorting and Nested Loops

This post presents my solutions to the [Perl Weekly Challenge 306](https://perlweeklychallenge.org/blog/perl-weekly-challenge-306/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 306 - Task 1 - Raku](#task1)
- [PWC 306 - Task 2 - Raku](#task2)
- [PWC 306 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 306 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 306 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 306 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 306 - Task 1 in PL/Java](#task1pljava)
- [PWC 306 - Task 2 in PL/Java](#task2pljava)
- [PWC 306 - Task 1 in Python](#task1python)
- [PWC 306 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 306 - Task 1 - Raku Implementation

Given an array of bits, slice them from left to right and report if the integer value of the obtained number is prime.

<br/>
<br/>
```raku
sub MAIN( *@bits where { @bits.elems == @bits.grep( * ~~ /<[01]>/ ).elems } ) {

    my @primes;
    for 0 ..^ @bits.elems {
		@primes.push: True and next if ( @bits[ 0 .. $_ ].join.parse-base( 10 ).is-prime );
		@primes.push: False;
    }

    @primes.join( ', ' ).say;
}

```
<br/>
<br/>


In short, I slice the array of `@bits`, `join` into a string and convert to a decimal number using `parse-base`, on which I apply `is-prime`.


<a name="task2"></a>
## PWC 306 - Task 2 - Raku Implementation

Given a dictionary of letters and an array of words, sort them according to the order of the first letter into the dictionary


<br/>
<br/>
```raku
sub MAIN( Str $dictionary, *@words ) {

    my @sorted;
    my @alien-dictionary = $dictionary.lc.comb( /<[a .. z]>/ );

    @sorted = @words.sort( { @alien-dictionary.grep( * ~~ $^a.lc.comb[0], :k ).first <=> @alien-dictionary.grep( * ~~ $^b.lc.comb[0], :k ).first } );
    @sorted.join( ', ' ).say;
}

```
<br/>
<br/>

I simply `sort` the incoming array by means of the indexes of the letters in the dictionary.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 306 - Task 1 - PL/Perl Implementation


Pretty much the same implementation as in Raku, with two main differences:
- I need to use `oct` to convert a manually crafted string `0b` from the array slicing;
- I need an internal functio `is_prime` to check if a number is prime.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc306.task1_plperl( bit[] )
RETURNS SETOF bool
AS $CODE$

   my ( $bits ) = @_;

   my $is_prime = sub {
      my ( $number ) = @_;
      return 0 if ( $number <= 1 );

      for ( 2 .. $number - 1  ) {
      	  return 0 if ( $number % $_ == 0 );
      }

      return 1;
   };

   for ( 0 .. $bits->@* - 1 ) {
       my $int = oct( '0b' .  join( '', $bits->@[ 0 .. $_ ] ) );
       return_next( $is_prime->( $int ) );
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 306 - Task 2 - PL/Perl Implementation

Schwartzian Transform to the rescue to sort the words depending on the value of the dictionary.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc306.task2_plperl( char[], text[] )
RETURNS SETOF text
AS $CODE$

   my ( $dictionary, $words ) = @_;

   my $alien_dictionary = {};
   my $index = 0;

   $alien_dictionary->{ lc $_ } = $index++ for ( $dictionary->@* );


   my @sorted =
		    map { $_->[ 1 ] }
		    sort { $a->[ 0 ] <=> $b->[ 0 ] }
		    map { [ $alien_dictionary->{ lc(  ( split //, $_ )[ 0 ] ) }, $_ ] }
		    $words->@*;

   return [ @sorted ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 306 - Task 1 - PL/PgSQL Implementation

Iterative approach, similar to the PL/Perl one.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION is_prime( n int )
RETURNS bool
AS $CODE$
DECLARE
	i int;
BEGIN
	IF n <= 1 THEN
	   RETURN false;
	END IF;

	FOR i IN 2 .. n - 1 LOOP
	    IF mod( n, i ) = 0 THEN
	       RETURN false;
	    END IF;
	END LOOP;

	RETURN true;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc306.task1_plpgsql( bits bit[] )
RETURNS SETOF bool
AS $CODE$
DECLARE
	i int;
	s text;
BEGIN

	FOR i in 1 .. array_length( bits, 1 ) LOOP
	    SELECT '0b' || string_agg( v::text, '' )
	    INTO s
	    FROM unnest( bits[ 0 : i ] ) v;

	    IF is_prime( s::int ) THEN
	       RETURN NEXT true;
	    ELSE
	      RETURN NEXT false;
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
## PWC 306 - Task 2 - PL/PgSQL Implementation

I cheat on this, and use the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc306.task2_plpgsql( dictionary char[], words text[] )
RETURNS SETOF text
AS $CODE$
   SELECT pwc306.task2_plperl( dictionary, words );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 306 - Task 1 - PostgreSQL PL/Java Implementation

I use an internal function to test if a number is prime, and convert every slice into a string to parse it to an integer.


<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc306",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final Boolean[] task1_pljava( String[] bits ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc306.task1_pljava" );

		List<Boolean> primes = new LinkedList<Boolean>();

		for ( int i = 1; i <= bits.length; i++ ) {
		    int value = Integer.parseInt( String.join( "", Arrays.copyOfRange( bits, 0, i ) ), 2 );
		    primes.add( is_prime( value ) );
		}

		Boolean result[] = new Boolean[ primes.size() ];
		for ( int i = 0; i < result.length; i++ )
		    result[ i ] = primes.get( i );


		return result;

    }


    private final static boolean is_prime( int n ) {
		if ( n <= 1 )
		    return false;

		for ( int i = 2; i < n; i++ )
		    if ( n % i == 0 )
				return false;

		return true;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 306 - Task 2 - PostgreSQL PL/Java Implementation

I use a dirty approach in here: a nested loop.
I iterate over the letters of the dictionary first, then search for all the words that begin with such letter.
The result is that I encounter the words in the order of the dictionary.


<br/>
<br/>
```java
    public static final String[] task2_pljava( String[] dictionary, String[] words ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc306.task2_pljava" );

		List<String> sorted = new LinkedList<String>();

		for ( int i = 0; i < dictionary.length; i++ ) {
		    for ( String w : words ) {
				if ( dictionary[ i ].toLowerCase().equals( w.toLowerCase().substring( 0, 1 ) ) )
					sorted.add( w );
		    }
		}

		String[] result = new String[ sorted.size() ];
		for ( int i = 0; i < result.length; i++ )
		    result[ i ] = sorted.get( i );

		return result;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 306 - Task 1 - Python Implementation

Same implementation as in PL/Perl.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    bits = list( map( int, args ) )
    primes = []

    def is_prime( n ):
        if n <= 1:
            return False

        for i in range( 2, n - 1 ):
            if n % i == 0:
                return False

        return True

    for i in range( 0, len( bits ) ):
        v = int( ''.join( list( map( str, bits[ 0 : i + 1 ] ) ) ), 2 )
        primes.append( is_prime( v ) )

    return primes


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 306 - Task 2 - Python Implementation

Same trick and nested loop as in PL/Java.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    dictionary = args[ 0 ].lower()
    words = args[ 1 : ]

    sorted_words = []

    for x in dictionary:
        for w in words:
            if w.lower()[ 0 ] == x:
                sorted_words.append( w )

    return sorted_words


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
