---
layout: post
title:  "Perl Weekly Challenge 273: quite easy!"
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

# Perl Weekly Challenge 273: quite easy!

This post presents my solutions to the [Perl Weekly Challenge 273](https://perlweeklychallenge.org/blog/perl-weekly-challenge-273/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 273 - Task 1 - Raku](#task1)
- [PWC 273 - Task 2 - Raku](#task2)
- [PWC 273 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 273 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 273 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 273 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 273 - Task 1 in PL/Java](#task1pljava)
- [PWC 273 - Task 2 in PL/Java](#task2pljava)
- [PWC 273 - Task 1 in Python](#task1python)
- [PWC 273 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 273 - Task 1 - Raku Implementation

The first task was about computing the percentage of occurrency of a given char in a given string.

<br/>
<br/>
```raku
sub MAIN( Str $string, Str $char where { $char.chars == 1 } ) {
    ( $string.comb.grep( * ~~ $char ).elems / $string.comb.elems * 100 ).Rat.round.say;
}

```
<br/>
<br/>

A one liner does suffice!


<a name="task2"></a>
## PWC 273 - Task 2 - Raku Implementation

See if a given string contains a `b` and not any `a` following the `b`.


<br/>
<br/>
```raku
sub MAIN( Str $string where { $string.chars > 0 } ) {
    'True'.say and exit if ( $string ~~ / b / && $string !~~ / b .* a / );
    'False'.say;

}

```
<br/>
<br/>

A double pattern matching solves the problem.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 273 - Task 1 - PL/Perl Implementation

Similar to the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc273.task1_plperl( text, text )
RETURNS int
AS $CODE$
   use POSIX;

   my ( $string, $char ) = @_;

   die "Pass a single char!" if ( length( $char ) != 1 );

   return 0 if $string !~ / $char /x;
   return POSIX::ceil( ( scalar( grep { $_ eq $char } split //, $string ) / scalar( split //, $string ) * 100 ) );

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

Note the usage of `POSIX::ceil` to make the percentage the desired integer value, and hence the need for the untrusted language.


<a name="task2plperl"></a>
## PWC 273 - Task 2 - PL/Perl Implementation

A double pattern matching solves the problem.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc273.task2_plperl( text )
RETURNS boolean
AS $CODE$

   my ( $string ) = @_;

   return 1 if ( $string =~ / b /x and $string !~ / b .* a /x );
   return 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 273 - Task 1 - PL/PgSQL Implementation

A single *nested* query does the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc273.task1_plpgsql( s text, c char )
RETURNS int
AS $CODE$

   WITH letters AS ( SELECT v FROM regexp_split_to_table( s, '' ) v )
   , matches AS ( SELECT v FROM letters v WHERE v = c )
   , c_l AS ( SELECT count(*) as x FROM letters )
   , c_m AS ( SELECT count(*) as x FROM matches )
   SELECT m.x::numeric / l.x::numeric * 100
   FROM c_l l , c_m m;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The casting between `numeric` and `int` is already ceiled.


<a name="task2plpgsql"></a>
## PWC 273 - Task 2 - PL/PgSQL Implementation

Another single query to apply the double pattern matching.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc273.task2_plpgsql( s text )
RETURNS boolean
AS $CODE$

   WITH has_b AS ( SELECT v FROM regexp_count( s, 'b' ) v )
   , has_b_without_trailing_a AS ( SELECT v FROM regexp_count( s, 'b.*a' ) v )
   SELECT CASE b.v WHEN NULL THEN false ELSE true END
   FROM has_b b, has_b_without_trailing_a a
   WHERE a.v = 0 AND b.v >= 1;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 273 - Task 1 - PostgreSQL PL/Java Implementation

Even if not required for this application, use the stream API to count the number of characters that are within the given string.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc273",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( String s, String c ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc273.task1_pljava" );

		if ( c.length() != 1 )
		    throw new SQLException( "Need a single char!" );


		final int[] values = new int[]{ 0, 0 };
		values[ 1 ] = s.length();
		Stream.of( s.split( "" ) )
		    .forEach( cc -> {
			if ( cc.equals( c ) )
			     values[ 0 ]++;
		    } );

		return (int) Math.round( (double) values[ 0 ] / values[ 1 ] * 100 );
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 273 - Task 2 - PostgreSQL PL/Java Implementation

Same as in other implementations: apply a double pattern matching.


<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc273",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final boolean task2_pljava( String s ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc273.task2_pljava" );

		Pattern p1 = Pattern.compile( "b" );
		Pattern p2 = Pattern.compile( "b.*a" );

		return p1.matcher( s ).find() && ! p2.matcher( s ).find();
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 273 - Task 1 - Python Implementation

Almost a one liner, with the same logic as in Perl.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    s = args[ 0 ]
    c = args[ 1 ]

    return round( len( list( filter( lambda x: x == c, s ) ) ) / len( s ) * 100 )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 273 - Task 2 - Python Implementation

A double pattern matching.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_2( args ):
    s = args[ 0 ]
    p1 = re.compile( 'b' )
    p2 = re.compile( 'b.*a' )
    return p1.search( s ) and not p2.search( s )




# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>


Here I spent too much time since I forgot that `match` anchors to the beginning of the string, while `search` does not.
