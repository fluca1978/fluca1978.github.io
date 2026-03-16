---
layout: post
title:  "Perl Weekly Challenge 365: regexps to rule them all!"
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

# Perl Weekly Challenge 365: regexps to rule them all!

This post presents my solutions to the [Perl Weekly Challenge 365](https://perlweeklychallenge.org/blog/perl-weekly-challenge-365/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 365 - Task 1 - Raku](#task1)
- [PWC 365 - Task 2 - Raku](#task2)
- [PWC 365 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 365 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 365 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 365 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 365 - Task 1 in PL/Java](#task1pljava)
- [PWC 365 - Task 2 in PL/Java](#task2pljava)
- [PWC 365 - Task 1 in Python](#task1python)
- [PWC 365 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 365 - Task 1 - Raku Implementation

The first task was about to convert a string into a numeric one substituting every letter to its position in the alphabet, then summing all the digits the number of times specified as an input argument.

<br/>
<br/>
```raku
sub MAIN( Str $string, Int $base ) {
    my @alphabet = 'a' .. 'z';
    my $number = $string.comb.map( -> $letter { @alphabet.grep( * ~~ $letter, :k ).first.Int + 1 } ).join;

    my $iteration = $base;
    while ( $iteration-- > 0 ) {
		$number = [+] $number.comb;
    }

    $number.say;

}

```
<br/>
<br/>


I use the `$number` variable to accumulate the first digitalization of the string, then I sum the combination of every digit.


<a name="task2"></a>
## PWC 365 - Task 2 - Raku Implementation

The second task was about to count the number of valid words within a sentence, with a wrong word being specialied.

<br/>
<br/>
```raku
sub MAIN( Str $sentence ) {
    my @tokens;
    for ( $sentence.split( /\s+/ ) ) {
		next if $_ ~~ / <[0..9]> /;
		next if $_ ~~ / <[A..Z]> '-' /;
		next if $_ ~~ / '-' <[A..Z]> /;
		next if $_ ~~ / <[!.,]> .+ /;

		@tokens.push: $_;
    }

    @tokens.elems.say;
}

```
<br/>
<br/>


I accumulate the right words into `@tokens` and then print out the size of the array.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 365 - Task 1 - PL/Perl Implementation

Same implementation of Raku, more verbose because I implement a `sum` inner function to sum all digits at every iteration.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc365.task1_plperl( text, int )
RETURNS int
AS $CODE$

   my ( $string, $base ) = @_;

   my $alphabet = {};
   my $index    = 1;
   for ( 'a' .. 'z' ) {
       $alphabet->{ $_ } = $index++;
   }


   my $sum = sub {
      my $v = 0;
      for ( split //, $_[0] ) {
         $v += $_;
      }

      return $v;
   };

   my $number = join '',
                map { $alphabet->{ $_ } }
                split //, $string;

   while ( $base-- > 0 ) {
   	 $number = $sum->( $number );
   }

   return $number;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 365 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc365.task2_plperl( text )
RETURNS int
AS $CODE$

   my ( $sentence ) = @_;
   my @tokens;

   for ( split /\s+/, $sentence ) {
       next if /[0-9]/;
       next if /[A-Z]\-/;
       next if /\-[A-Z]/;
       next if /[!.,].+/;

       push @tokens, $_;
   }

   return scalar @tokens;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 365 - Task 1 - PL/PgSQL Implementation

I use regular expressions as in PL/Perl.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc365.task1_plpgsql( s text, b int )
RETURNS int
AS $CODE$
DECLARE
	current_sum int := 0;
	n int;
	c char;
	final_sum int := 0;
BEGIN

	FOR c IN SELECT v FROM regexp_split_to_table( s, '' ) v LOOP
	    final_sum := final_sum + ascii( c ) - ascii( 'a' ) + 1;
	END LOOP;


	WHILE b > 0 LOOP
	      current_sum := 0;
	      FOR n IN SELECT v::int FROM regexp_split_to_table( final_sum::text, '' ) v LOOP
	      	  current_sum := current_sum + n;
	      END LOOP;
	      b := b - 1;
	      final_sum := current_sum;
	END LOOP;

	RETURN current_sum;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

There is the need to change between integers and strings during all the steps.


<a name="task2plpgsql"></a>
## PWC 365 - Task 2 - PL/PgSQL Implementation

Here a simple SQL query does suffice!


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc365.task2_plpgsql( s text )
RETURNS int
AS $CODE$
   SELECT count(*)
   FROM   regexp_split_to_table( s, '\s+' ) v
   WHERE  v !~ '[0-9]'
   AND    v !~ '[A-Z]\-'
   AND    v !~ '\-[A-Z]'
   AND    v !~ '[!.,].+'
   ;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 365 - Task 1 - PostgreSQL PL/Java Implementation

I use the Stream API to do the computation.

<br/>
<br/>
```java
  @Function( schema = "pwc365",
             onNullInput = RETURNS_NULL,
             effects = IMMUTABLE )
  public static final int task1_pljava( String string, int base ) throws SQLException {
    logger.log( Level.INFO, "Entering pwc365.task1_pljava" );

    int result = Integer.parseInt( string.chars()
                                   .map( c -> c - 'a' + 1 )
                                   .mapToObj( String::valueOf )
                                   .collect( Collectors.joining() ) );

    while ( base-- > 0 ) {
      result = (int) String.valueOf( result ).chars().map( c -> c - '0' ).sum();
    }

    return result;
  }
```
<br/>
<br/>


I compute into `result` the first digitalization, then I iterate converting it into `String`, getting the `chars()` and mapping with the difference between `0`. The trick here is that a char is converted into a number and substrcting the initial char (either `a` or `0`) gives me a numeric value.



<a name="task2pljava"></a>
## PWC 365 - Task 2 - PostgreSQL PL/Java Implementation

Same implementation as in PL/Perl but with a very annoying thing.


<br/>
<br/>
```java
  public static final int task2_pljava( String sentence ) throws SQLException {
    logger.log( Level.INFO, "Entering pwc365.task2_pljava" );

    List<String> tokens = new LinkedList<String>();

    for ( String s : sentence.split( "\\s+" ) ) {
      if ( s.matches( ".*\\d+.*" )
           || s.matches( ".*[A-Z]-.*" )
           || s.matches( ".*-[A-Z].*" )
           || s.matches( ".*[!.,].+") )
        continue;

      tokens.add( s );
    }

    return tokens.size();
  }
```
<br/>
<br/>

Since I don't want to use the whole regular expression package, I use `String.matches()` which compares with the full string, so it will not match parts of the string. This is why `"[!.,].+"` does not work and requires something on the left side too as in `".*[!.,].+"`.


# Python Implementations

<a name="task1python"></a>
## PWC 365 - Task 1 - Python Implementation

Similar to PL/PgSQL implementation, there is the annoying need to convert from `int` to `str` at different stages.

<br/>
<br/>
```python
import sys
import string

# task implementation
# the return value will be printed
def task_1( args ):
    word = args[ 0 ]
    base = int( args[ 1 ] )

    alphabet = list( string.ascii_lowercase )

    result = ""
    for v in word:
        result += str( alphabet.index( v ) + 1 )

    summy = 0
    current = 0
    for v in result:
        summy += int( v )

    while base > 0:
        current = 0
        for v in str( summy ):
            current += int( v )

        summy = current
        base  = base - 1


    return summy

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 365 - Task 2 - Python Implementation

I use regular expressions to exclude words that do not satisfy the needs.


<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_2( args ):
    sentence = args[ 0 ]

    counter = 0

    for word in sentence.split( ' ' ):
        if re.search( r'\d+', word ) or re.search( r'[A-Z]-', word ) or re.search( r'-[A-Z]', word ) or re.search(  r'[!.,].+', word ):
            continue
        counter += 1

    return counter



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
