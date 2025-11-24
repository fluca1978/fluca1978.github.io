---
layout: post
title:  "Perl Weekly Challenge 349: moving and grepping"
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

# Perl Weekly Challenge 349: moving and grepping

This post presents my solutions to the [Perl Weekly Challenge 349](https://perlweeklychallenge.org/blog/perl-weekly-challenge-349/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 349 - Task 1 - Raku](#task1)
- [PWC 349 - Task 2 - Raku](#task2)
- [PWC 349 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 349 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 349 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 349 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 349 - Task 1 in PL/Java](#task1pljava)
- [PWC 349 - Task 2 in PL/Java](#task2pljava)
- [PWC 349 - Task 1 in Python](#task1python)
- [PWC 349 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 349 - Task 1 - Raku Implementation

The first task was to analyze a given string to find out the longest substring made by only the same repeated char.

<br/>
<br/>
```raku
sub MAIN( Str $string ) {

    my %powers;
    for $string.comb -> $l {
		%powers{ ( $string ~~ / $l+ / ).Str.chars } = $l;
    }

    %power.keys.max.say;
    #(%powers{ %powers.keys.max } xx %powers.keys.max).join.say;
}

```
<br/>
<br/>

I iterate over all the single chars and place into a `%powers` hash keyed by the length of the substring. Then I extract the max key.
Note the commented out line, that prints the longest string found.


<a name="task2"></a>
## PWC 349 - Task 2 - Raku Implementation

Given a string that reports single movements like `U`p, `D`own, `L`eft and `R`ight, find out if the final result is to pass for the center (origin).


<br/>
<br/>
```raku
sub MAIN( Str $directions ) {
    my %moves;
    %moves{ $_ }++ for ( $directions.comb );

    'True'.say and exit if ( %moves<U> == %moves<D> && %moves<R> == %moves<L> );
    'False'.say;
}

```
<br/>
<br/>

I count every movement in any direction keeping the `%moves` hash, and then see if the movements in opposite directions have the same count, that is they move to the center.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 349 - Task 1 - PL/Perl Implementation

Similar implementation to Raku, using backreference.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc349.task1_plperl( text )
RETURNS int
AS $CODE$

   my ( $string ) = @_;
   my $longest = 0;

    while ( $string =~ / (.) \1+ /xg ) {
        $longest = length( $& ) if ( length( $& ) > $longest );
    }


   return $longest;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 349 - Task 2 - PL/Perl Implementation

Same implementation as in Raku, using an hash to compute the moves in any direction.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc349.task2_plperl( text )
RETURNS boolean
AS $CODE$

   my ( $directions ) = @_;

   my $moves = {};
   for ( split //, $directions ) {
       $moves->{ $_ }++;
   }

   return 1 if (  $moves->{ 'D' } == $moves->{ 'U' }
               && $moves->{ 'L' } == $moves->{ 'R' } );
  return 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 349 - Task 1 - PL/PgSQL Implementation

Here I use a nested loop.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc349.task1_plpgsql( s text )
RETURNS int
AS $CODE$
DECLARE
	c text;
	r int := 0;
	ss text;
	cc record;
BEGIN

	FOREACH c IN ARRAY regexp_split_to_array( s, '' ) LOOP
		FOR cc IN SELECT v FROM regexp_matches( s, c || '+' ) v LOOP
			IF length( cc.v::text ) - 2 > r THEN
			   r = length( cc.v::text ) - 2;
			END IF;
		END LOOP;
	END LOOP;

	RETURN r;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

First I split the string into the chars, and then search for a string made by repetitions of the same char. The result is a `text[]`, so when stringigfied it includes two more chars (the brackets), hence the effective string length is subtracted by two.



<a name="task2plpgsql"></a>
## PWC 349 - Task 2 - PL/PgSQL Implementation

A single query does suffice to solve the problem.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc349.task2_plpgsql( directions text )
RETURNS boolean
AS $CODE$

   WITH moves AS (
   	SELECT v, count( * ) as c
	FROM regexp_split_to_table( directions, '' ) v
	GROUP BY 1
   )
   , summary AS (
       SELECT m1.c - m2.c AS h, m3.c - m4.c AS v
       FROM moves m1, moves m2, moves m3, moves m4
       WHERE
       m1.v = 'L' AND m2.v = 'R'
       AND m3.v = 'U' AND m4.v = 'D'
    )
    SELECT exists (
    	   SELECT *
	   FROM summary
	   WHERE h = 0 AND v = 0
    );


$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The `moves` CTE coutns all the moves in any direction, while the `summary` does the horizontal and vertical. Last, if the summary has zero-ed rows, we are fine and the `select exists` returns a `true` value.


# Java Implementations

<a name="task1pljava"></a>
## PWC 349 - Task 1 - PostgreSQL PL/Java Implementation

I use a very trivial approach: I simply iterate over every char and see if the previous one is equal to the current one, in such case I keep counting and trak the max length of the substring.


<br/>
<br/>
```java
    @Function( schema = "pwc349",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task1_pljava( String string ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc349.task1_pljava" );

		int result = 0;
		String previous = null;
		int max = 0;

		for ( String current : string.split( "" ) ) {
		    if ( previous == null || ! previous.equals( current ) ) {
				previous = current;

				if ( result > max )
				    max = result;

				result = 0;
		    }


		    result++;
		}

		if ( result > max )
		    max = result;


		return max;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 349 - Task 2 - PostgreSQL PL/Java Implementation

Even simpler than the previous implementations: I count the movements in the four directions and then compare the opposite directions.

<br/>
<br/>
```java
    @Function( schema = "pwc349",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final boolean task2_pljava( String directions ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc349.task2_pljava" );

		int left = 0, right = 0, up = 0, down = 0;

		for ( String m : directions.split( "" ) ) {
		    if ( m.equals( "L" ) )
				left++;
		    else if ( m.equals( "R" ) )
				right++;
		    else if ( m.equals( "U" ) )
				up++;
		    else if ( m.equals( "D" ) )
				down++;
		}


		return left == right && up == down;


    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 349 - Task 1 - Python Implementation

Same implementation as in PL/Java.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    string = args[ 0 ]
    previous = ''
    result   = 0
    count    = 0

    for current in string :
        if previous is None or previous != current :
            if count > result :
                result = count

            count = 0
            previous = current

        count += 1

    if count > result :
        result = count

    return result



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 349 - Task 2 - Python Implementation

Simpler than PL/Java, I simply keep the main directions `h`orizintal and `v`ertical and ensure they are both zero at the end.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    h = 0
    v = 0

    for move in args[ 0 ]:
        if move == 'L' :
            h -= 1
        elif move == 'R':
            h += 1
        elif move == 'U':
            v += 1
        elif move == 'D':
            v -= 1

    return v == 0 and h == 0


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
