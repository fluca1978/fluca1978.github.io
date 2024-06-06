---
layout: post
title:  "Perl Weekly Challenge 272: Quick and Simple"
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

# Perl Weekly Challenge 272: Quick and Simple

This post presents my solutions to the [Perl Weekly Challenge 272](https://perlweeklychallenge.org/blog/perl-weekly-challenge-272/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 272 - Task 1 - Raku](#task1)
- [PWC 272 - Task 2 - Raku](#task2)
- [PWC 272 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 272 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 272 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 272 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 272 - Task 1 in PL/Java](#task1pljava)
- [PWC 272 - Task 2 in PL/Java](#task2pljava)
- [PWC 272 - Task 1 in Python](#task1python)
- [PWC 272 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 272 - Task 1 - Raku Implementation

The first task was to convert an IPv4 like string replacing the dots with `[.]`.

<br/>
<br/>
```raku
sub MAIN( Str $ip is copy
	  where { $ip ~~ /^ ( \d ** 1..3 '.' ) ** 3 \d ** 1..3  / } ) {

    $ip .= subst( '.', '[.]', :g );
    $ip.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 272 - Task 2 - Raku Implementation

The second task was about computing the sum of the differences between an ASCII value for a char and its following char.


<br/>
<br/>
```raku
sub MAIN( Str $string where { $string.elems > 0  } ) {
    my $score = 0;
    my @letters = $string.comb;

    for 0 ..^ @letters.elems - 1 -> $index {
		$score += abs( @letters[ $index ].Str.ord - @letters[ $index + 1 ].Str.ord );
    }

    $score.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 272 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, I use a regular expression substitution.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc272.task1_plperl( text )
RETURNS text
AS $CODE$

   my ( $ip ) = @_;

   die "IP not valid" if ( $ip !~ / ^ ( \d {1,3} [.] ) {3} \d {1,3} $ /x );

   $ip =~ s/[.]/\[\.\]/g;
   return $ip;

$CODE$
LANGUAGE plperl;
```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 272 - Task 2 - PL/Perl Implementation

Same approach as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc272.task2_plperl( text )
RETURNS int
AS $CODE$

   my ( $string ) = @_;

   my $score = 0;
   my @chars = split //, $string;

   for my $index ( 0 .. $#chars - 1 ) {
      my $diff = ord( $chars[ $index ] ) - ord( $chars[ $index + 1 ] );
      $score += $diff > 0 ? $diff : $diff * -1;
   }

   return $score;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 272 - Task 1 - PL/PgSQL Implementation

Here a single query can replace globally with a regular expression.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc272.task1_plpgsql( addr text )
RETURNS text
AS $CODE$
   SELECT
   regexp_replace( addr, '\.', '[.]', 'g' );
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 272 - Task 2 - PL/PgSQL Implementation

Same approach as in PL/Perl, with a loop to accumulate the score.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc272.task2_plpgsql( s text )
RETURNS int
AS $CODE$
DECLARE
	score int;
	previous int;
	v int;
BEGIN

	previous := NULL;
	score := 0;
	FOR v IN SELECT ascii( x )::int FROM regexp_split_to_table( s, '' ) x LOOP
	    IF previous IS NOT NULL THEN
	       score := score + abs( previous - v::int );
	    END IF;

	    previous := v::int;

	END LOOP;

	RETURN score;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

To make it simpler to know the previous character value, I store it in a ad-hoc variable `previous`.

# Java Implementations

<a name="task1pljava"></a>
## PWC 272 - Task 1 - PostgreSQL PL/Java Implementation

A one liner!

<br/>
<br/>
```java
 public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc272",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task1_pljava( String ip ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc272.task1_pljava" );

	   return ip.replaceAll( "\\.", "[.]" );
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 272 - Task 2 - PostgreSQL PL/Java Implementation

Same approach as in PL/Perl.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc272",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( String string ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc272.task2_pljava" );

		int score = 0;

		for ( int i = 0; i < string.length() - 1; i++ ) {
		    int diff = string.charAt( i ) - string.charAt( i + 1 );
		    score += diff > 0 ? diff : diff * -1;
		}

		return score;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 272 - Task 1 - Python Implementation

Yay! My first one liner in Python (or almost)!
<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    return args[ 0 ].replace( '.', '[.]' )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 272 - Task 2 - Python Implementation

Same approach as in PL/Java.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    score    = 0
    string   = args[ 0 ]
    previous = None

    for x in string:
        if previous is not None:
            diff = ord( x ) - ord( previous )
            if diff < 0:
                diff *= -1

            score += diff

        previous = x

    return score


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
