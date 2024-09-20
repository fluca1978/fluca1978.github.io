---
layout: post
title:  "Perl Weekly Challenge 287: in regexp we trust!"
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

# Perl Weekly Challenge 287: in regexp we trust!

This post presents my solutions to the [Perl Weekly Challenge 287](https://perlweeklychallenge.org/blog/perl-weekly-challenge-287/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 287 - Task 1 - Raku](#task1)
- [PWC 287 - Task 2 - Raku](#task2)
- [PWC 287 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 287 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 287 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 287 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 287 - Task 1 in PL/Java](#task1pljava)
- [PWC 287 - Task 2 in PL/Java](#task2pljava)
- [PWC 287 - Task 1 in Python](#task1python)
- [PWC 287 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 287 - Task 1 - Raku Implementation

Given a string representing a password, ensures it passes a few strong checks.

<br/>
<br/>
```raku
sub MAIN( Str :$password ) {

    my $ok = True;
    'Too short'.say and $ok = False if ( $password.chars < 6 );
    'At least one lowercase char'.say and $ok = False if ( $password !~~ / <[a .. z]> / );
    'At least one uppercase char'.say and $ok = False if ( $password !~~ / <[A .. Z]> / );
    'At least one digit char'.say and $ok = False if ( $password !~~ / <[0 .. 9]> / );
    'No three repetitions in a row'.say and $ok = False if ( $password ~~ / (<[a..zA..Z0..9]>)$0$0  / );


    'Strong enough'.say if ( $ok );
    'Weak!'.say if ( ! $ok );
    exit( $ok ?? 0 !! 1 );
}

```
<br/>
<br/>


I use only regular expressions to check the password strength, while it would be possible to aggregate all the regular expressions in one, having them separated allows me to provide some sort of hint to the user.




<a name="task2"></a>
## PWC 287 - Task 2 - Raku Implementation

Given a string that represent a number, with optionally a sign, decimal digits and exponential part, see if the string represent a good number.

<br/>
<br/>
```raku
sub MAIN( Str $number ) {
    'False'.say and exit( 1 ) if ( $number !~~ / ^ <[+-]>? \d+ (.\d+)? (E<[+-]>?\d+)? $ / );
    'True'.say;

}

```
<br/>
<br/>

A single regular expression does suffice to the trick!

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 287 - Task 1 - PL/Perl Implementation

Same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc287.task1_plperl( text )
RETURNS bool
AS $CODE$

   my ( $password ) = @_;
   return 0 if ( length( $password ) < 6 );
   return 0 if ( $password !~ / [a-z] /x );
   return 0 if ( $password !~ / [A-Z] /x );
   return 0 if ( $password !~ / [0-9] /x );
   return 0 if ( $password ~~ / (.)\1\1 /x );

   return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 287 - Task 2 - PL/Perl Implementation

Same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc287.task2_plperl( text )
RETURNS bool
AS $CODE$

   my ( $number ) = @_;

   return 1 if ( $number =~ / ^ [+-]? \d+ (.\d+)? (E[+-]?\d+)? $/x );
   return 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 287 - Task 1 - PL/PgSQL Implementation

Here I need a few queries to count how many matches a regular expression finds against the given string.
The last check, on the other hand, ensures the regular expression does not match at all.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc287.task1_plpgsql( pwd text )
RETURNS bool
AS $CODE$
DECLARE
	counter int := 0;
BEGIN

	IF length( pwd ) < 6 THEN
	   RETURN false;
	END IF;

	SELECT count( x )
	INTO counter
	FROM regexp_matches( pwd, '[a-z]', 'g' ) x;

	IF counter <= 0 THEN
	   RETURN false;
	END IF;

	SELECT count( x )
	INTO counter
	FROM regexp_matches( pwd, '[A-Z]', 'g' ) x;

	IF counter <= 0 THEN
	   RETURN false;
	END IF;

	SELECT count( x )
	INTO counter
	FROM regexp_matches( pwd, '[0-9]', 'g' ) x;

	IF counter <= 0 THEN
	   RETURN false;
	END IF;


	SELECT count( x )
	INTO counter
	FROM regexp_matches( pwd, '(.)\1\1', 'g' ) x;

	IF counter > 0 THEN
	   RETURN false;
	END IF;


	RETURN true;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 287 - Task 2 - PL/PgSQL Implementation

Similar approach to the solution of the task one: I count the matches of the regular expression to see if there is an error or not.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc287.task2_plpgsql( n text )
RETURNS bool
AS $CODE$
DECLARE
	counter int := 0;
BEGIN

	SELECT count( x )
	INTO counter
	FROM regexp_matches( n, '^[+-]?\d+(.\d+)?(E[+-]?\d+)?$' ) x;

	IF counter > 0 THEN
	   RETURN true;
    ELSE
	  RETURN false;
    END IF;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 287 - Task 1 - PostgreSQL PL/Java Implementation

Similarly to the Raku and PL/Perl implementations, use a few regular expressions and see if they match (and the last one must not match).


<br/>
<br/>
```java
public static final boolean task1_pljava( String password ) throws SQLException {
	if ( password.length() < 6 )
	    return false;

	Pattern lowerCase   = Pattern.compile( "[a-z]" );
	Pattern upperCase   = Pattern.compile( "[A-Z]" );
	Pattern digit       = Pattern.compile( "[0-9]" );
	Pattern repetitions = Pattern.compile( "(.)\\1\\1" );

	return lowerCase.matcher( password ).find()
	    && upperCase.matcher( password ).find()
	    && digit.matcher( password ).find()
	    && ! repetitions.matcher( password ).find();
}
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 287 - Task 2 - PostgreSQL PL/Java Implementation

Again, a single regular expression does the trick.

<br/>
<br/>
```java
public static final boolean task2_pljava( String number ) throws SQLException {
	Pattern numberRegexp = Pattern.compile( "^[+-]?\\d+([.]\\d+)?(E[+-]?\\d+)?$" );
	return numberRegexp.matcher( number ).find();
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 287 - Task 1 - Python Implementation

Apply all the regular expressions one after the other, but keep in mind the usage of `search` instead of `find`.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    password = args[ 0 ]
    lc    = re.compile( '[a-z]' )
    uc    = re.compile( '[A-Z]' )
    dg    = re.compile( '[0-9]' )
    wrong = re.compile( '(.)\\1\\1' )
    return lc.search( password ) and uc.search( password ) and dg.search( password ) and wrong.search( password ) is None

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 287 - Task 2 - Python Implementation

A single regular expression to solve the problem.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_2( args ):
    number = args[ 0 ]
    good = re.compile( '^[+-]?\\d+([.]\\d+)?(E[+-]?\\d+)?$' )
    return good.match( number ) is not None

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
