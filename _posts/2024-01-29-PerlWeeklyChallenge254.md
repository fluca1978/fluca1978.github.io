---
layout: post
title:  "Perl Weekly Challenge 254: vowels and roots"
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

# Perl Weekly Challenge 254: vowels and roots

This post presents my solutions to the [Perl Weekly Challenge 254](https://perlweeklychallenge.org/blog/perl-weekly-challenge-254/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 254 - Task 1 - Raku](#task1)
- [PWC 254 - Task 2 - Raku](#task2)
- [PWC 254 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 254 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 254 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 254 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 254 - Task 1 in PL/Java](#task1pljava)
- [PWC 254 - Task 2 in PL/Java](#task2pljava)
- [PWC 254 - Task 1 in Python](#task1python)
- [PWC 254 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 254 - Task 1 - Raku Implementation

The first task was about finding out if a given number was a power of three of an integer.

<br/>
<br/>
```raku
sub MAIN( Int $n ) {
    #    say ( $n ** ( 1 / 3 ) ).Int == ( $n ** ( 1 / 3 ) );
    for ( 2 ..^ $n.sqrt.Int ) {
		'true'.say and exit if ( $_ ** 3 == $n );
    }

    'false'.say;
}

```
<br/>
<br/>

In the beginning I thought to test if the number, powered at `1/3` resulted in an integer, but since there is `Rat` in the middle, some approximations could ruin the final result. So I took an iterative approach iterating on a every sensible number seeing if it can result in the power of three equal to the given number,



<a name="task2"></a>
## PWC 254 - Task 2 - Raku Implementation

The seconda task was about reversing all the vowels in a given string, so that the last (rightmost) one becomes the first (leftmost) and so on.

<br/>
<br/>
```raku
sub MAIN( Str $word ) {
    my @reversed;
    my @vowels.push: |$word.lc.comb.grep( * ~~ / <[aeiou]> / ).reverse;

    for $word.comb {
	   @reversed.push( $_ ) and next if ( $_.lc !~~ / <[aeiou]> / || @vowels.elems == 0 );
	   @reversed.push: @vowels.shift if ( @vowels.elems > 0 );
    }

    @reversed.join.say;
}

```
<br/>
<br/>

The idea is to take a `@vowels` array to be used as a stack, so I place all the vowels in a reversed order. Then I loop thru every letter of the original word, and in the case the current letter is not a vowel or the `@vowels` is empty, I simply append such letter, otherwise it is a vowel and I pick the first one I can find.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 254 - Task 1 - PL/Perl Implementation

Same approach as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc254.task1_plperl( int )
RETURNS bool
AS $CODE$

   my ( $num ) = @_;

   for ( 2 .. ( $num / 2 ) ) {
       return 1 if ( $_ ** 3 == $num );
   }

   return 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 254 - Task 2 - PL/Perl Implementation

Same approach as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc254.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $word ) = @_;

   my @vowels = reverse grep { $_ =~ / [aeiou] /ix } split( //, $word );
   my $output = '';

   for ( split //, $word ) {
       $output .= $_ and next if ( $_ !~ / [aeiou] /ix || ! @vowels );
       $output .= shift( @vowels ) and next;
   }

   return $output;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 254 - Task 1 - PL/PgSQL Implementation

Same approach as in PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc254.task1_plpgsql( n int )
RETURNS bool
AS $CODE$
BEGIN
	FOR i IN 2 .. sqrt( n )::int LOOP
	    IF pow( i, 3 ) = n THEN
	       RETURN TRUE;
	    END IF;
	END LOOP;

	RETURN FALSE;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 254 - Task 2 - PL/PgSQL Implementation

Here I use a temporary table as a stack: I place all the vowels into the table and then extract one at a time by ordering by the insertion number in descending order, deleting then the letter.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc254.task2_plpgsql( word text )
RETURNS text
AS $CODE$
DECLARE
	output_string text := '';
	current_vowel char;
	current_index int;
	remaining_vowels int;
	letter char;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS vowels( v char, i serial );
	TRUNCATE vowels;

	INSERT INTO vowels( v )
	SELECT v
	FROM regexp_split_to_table( lower( word ), '' ) v
	WHERE v IN ( 'a', 'e', 'i', 'o', 'u' );

	FOR letter IN SELECT v FROM regexp_split_to_table( lower( word ), '' ) v LOOP

	    SELECT count( * )
	    FROM vowels
	    INTO remaining_vowels;

	    IF letter NOT IN ('a', 'e', 'i', 'o', 'u' ) OR remaining_vowels = 0 THEN
	       output_string := output_string || letter;
	    ELSE
			SELECT v, i
			INTO current_vowel, current_index
			FROM vowels
			ORDER BY i DESC;

			output_string := output_string || current_vowel;
			DELETE FROM vowels WHERE i = current_index;
	    END IF;
	END LOOP;

	RETURN output_string;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 254 - Task 1 - PostgreSQL PL/Java Implementation

The implementation is as in PL/Perl.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
    public static final boolean task1_pljava( int num ) throws SQLException {
		for ( int i = 2; i < Math.sqrt( num ); i++ )
		    if ( Math.pow( i, 3 ) == num )
				return true;
		

		return false;
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 254 - Task 2 - PostgreSQL PL/Java Implementation

Here I use an utility method `isVowel` to compact the code and test if a given letter is a vowel. Then, the approach is similar to PL/perl.


<br/>
<br/>
```java
    public static final String task2_pljava( String word ) throws SQLException {
		String result = "";
		Stack<String> vowels = new Stack<String>();

		for ( String letter : word.split( "" ) ) {
		    if ( isVowel( letter ) )
				vowels.push( letter );
		}

		for ( String letter : word.split( "" ) ) {
		    if ( ! isVowel( letter ) || vowels.empty() )
				result += letter;
		    else
				result += vowels.pop();
		}

		return result;
    }


	public static final boolean isVowel( String letter ) {
	    return letter.toLowerCase().equals( "a" )
		|| letter.toLowerCase().equals( "e" )
		|| letter.toLowerCase().equals( "i" )
		|| letter.toLowerCase().equals( "o" )
		|| letter.toLowerCase().equals( "u" );
	}
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 254 - Task 1 - Python Implementation

Same implementatin as in PL/Perl.

<br/>
<br/>
```python
import sys
import math

# task implementation
# the return value will be printed
def task_1( args ):
    num = int( args[ 0 ] )
    for i in range( 2, int( math.sqrt( num ) ) ):
        if ( i ** 3 ) == num:
            return True

    return False


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 254 - Task 2 - Python Implementation

Same idea as in PL/Perl, but note how difficult it is to get the reversed list of vowels!

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    word = args[ 0 ].lower()
    vowels =  list( reversed( list( filter( lambda x: x in ('a','e','i','o','u'), word ) ) ) )
    output = ''
    for letter in word:
        if letter not in ( 'a', 'e', 'i', 'o', 'u' ) or len( vowels ) == 0:
            output += letter
        else:
            output += vowels.pop( 0 )

    return output



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
