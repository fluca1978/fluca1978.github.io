---
layout: post
title:  "Perl Weekly Challenge 246: Brute Force Math!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 246: Brute Force Math!

This post presents my solutions to the [Perl Weekly Challenge 246](https://perlweeklychallenge.org/blog/perl-weekly-challenge-246/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 246 - Task 1 - Raku](#task1)
- [PWC 246 - Task 2 - Raku](#task2)
- [PWC 246 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 246 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 246 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 246 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 246 - Task 1 in Python](#task1python)
- [PWC 246 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 246 - Task 1 - Raku Implementation

The first task was about extracting six random numbers (without repetitions) less than `49`.

<br/>
<br/>
```raku
sub MAIN() {

    my @lottery;
    while ( @lottery.elems < 6 ) {
		@lottery.push: $_ if ( ! @lottery.grep( $_ ) ) given ( 49.rand.Int );
    }

    @lottery.join( "\n" ).say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 246 - Task 2 - Raku Implementation

The second task was about finding out, given an input array of five integers, if the array represents a *linear recurrence of second order*.
Since I'm not good at this kind of math, I implemented it with a *brute force* approach where I scan from `1` to `infinity` the possibiliies and exits as soon as I'm sure I've found an answer.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == 5 && @nums.grep( * ~~ Int ).elems == @nums.elems }  ) {

    for 2 ..^ @nums.elems {
	# a[n] = p * a[n-2] + q * a[n-1] with n > 1

	my $ok = False;
	for 1 .. Inf -> $p {
	    for 1 .. Inf -> $q {
				if ( @nums[ $_ ] == ( $p * @nums[ $_ - 2 ] + $q * @nums[ $_ - 1 ] )
					|| @nums[ $_ ] == ( $p * -1 * @nums[ $_ - 2 ] + $q * @nums[ $_ - 1 ] )
					|| @nums[ $_ ] == ( $p * -1 * @nums[ $_ - 2 ] + $q * -1 * @nums[ $_ - 1 ] )
					|| @nums[ $_ ] == ( $p * @nums[ $_ - 2 ] + $q * -1 * @nums[ $_ - 1 ] ) ) {
				    $ok = True;
				    last;
				}
	    }

	    last if $ok;
	}


       'False'.say and exit if  ! $ok;
    }

    'True'.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 246 - Task 1 - PL/Perl Implementation

Same approach as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc246.task1_plperl()
RETURNS SETOF int
AS $CODE$
   my @lottery;

   while ( @lottery < 6 ) {
   	 my $value = int ( rand( 49 ) );
	 if ( ! grep( { $_ == $value } @lottery ) ) {
	    push @lottery, $value;
	    return_next( $value );
	 }
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 246 - Task 2 - PL/Perl Implementation

Same *brute force* approach as in Raku.

Note that I need to use a C-style `for` loop to simulate the infinity upper boundary, since `inf` from `bigint` is not usable in a range.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc246.task2_plperl( int[] )
RETURNS bool
AS $CODE$
   my ( $nums ) = @_;

   die "Need 5 numbers" if ( $nums->@* != 5 );
   my $ok = 0;
   my $ko = $nums->@* - 2;

   for ( 2 .. $nums->@* - 1 ) {
       for ( my $p = 1; ; $p++ ) {
       	   for ( my $q = 1; ; $q++ ) {
	      if ( $nums->@[ $_ ] == ( $p * $nums->[ $_ - 2 ] + $q * $nums->[ $_ - 1 ] )
	         || $nums->@[ $_ ] == ( $p * -1 * $nums->[ $_ - 2 ] + $q * $nums->[ $_ - 1 ] )
	         || $nums->@[ $_ ] == ( $p * -1 * $nums->[ $_ - 2 ] + $q * -1 * $nums->[ $_ - 1 ] )
	         || $nums->@[ $_ ] == ( $p * $nums->[ $_ - 2 ] + $q * -1 * $nums->[ $_ - 1 ] ) ) {

	         $ok = 1;
		 $ko--;
	         last;
	      }
          }

          last if $ok;
      }
   }

   return ( $ko == 0 );
$CODE$
LANGUAGE plperl;
```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 246 - Task 1 - PL/PgSQL Implementation

I use a couple of tricks here, that surely do not make this solution a good one given that I need only six values, but allows me to explain some capabilities of PostgreSQL.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc246.task1_plpgsql()
RETURNS SETOF INT
AS $CODE$
DECLARE
	counting int;
	current_value int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS lottery( v int, CHECK( v <= 49 ), PRIMARY KEY( v ) );
	TRUNCATE lottery;

	counting := 0;
	WHILE counting < 6 LOOP
	      current_value := ( random() * 100 )::int;

	      IF current_value <= 49 THEN
	      	      INSERT INTO lottery( v )
	      	      VALUES( current_value )
		      ON CONFLICT ( v ) DO NOTHING;
	      END IF;


	      SELECT count(*)
	      INTO counting
	      FROM lottery;

	END LOOP;

	RETURN QUERY
	SELECT * FROM lottery;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


I use a temporary table `lottery` to store unique values that I'm going to generate randomly.
The idea is while `counting`, that contains the number of rows in the temporary table, is less than `6`, I'm going to generate a new random value by means of `random` (multiplied by `100` to ensure it gives me a number between `1` and `100`). If the genrrated value if less than `49` I insert it into the temporary table, *skipping the insert in the case the number is already there*!
Once the table has been filled with six rows, I can return all of them with a simple query.


<a name="task2plpgsql"></a>
## PWC 246 - Task 2 - PL/PgSQL Implementation

Same brute force approach shown before.

Note that here I simulate a quite infinity upper boud with a `99999` fixed value, but it would be possible to use an always increasing loop.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc246.task2_plpgsql( nums int[] )
RETURNS bool
AS $CODE$
DECLARE
	current_index int;
	p int;
	q int;
	ok bool := false;
	ko int;
BEGIN

	ko := array_length( nums, 1 ) - 3 + 1;

	FOR current_index IN 3 .. array_length( nums, 1 ) LOOP
	    ok := false;

	    FOR p IN 1 .. 99999 LOOP
	    	FOR q IN 1 .. 9999 LOOP
		    IF nums[ current_index ] = ( p * nums[ current_index - 2 ] + q * nums[ current_index - 1 ] )
		       OR nums[ current_index ] = ( p * -1 * nums[ current_index - 2 ] + q * nums[ current_index - 1 ] )
		       OR nums[ current_index ] = ( p * -1 * nums[ current_index - 2 ] + q * -1 *nums[ current_index - 1 ] )
		       OR nums[ current_index ] = ( p * nums[ current_index - 2 ] + q * -1 * nums[ current_index - 1 ] ) THEN
		       	  ok := true;
			  ko := ko - 1;
			  EXIT;
	            END IF;
		END LOOP;

		IF ok THEN
		   EXIT;
		END IF;

	    END LOOP;
	END LOOP;

	RETURN ko = 0;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 246 - Task 1 - Python Implementation

A very simple implementation, using the random generation and an array to store the values so that I can check if I've already got a value before.

<br/>
<br/>
```python
import sys
import random

# task implementation
def main( argv ):
    values = []
    while len( values ) < 6:
        current_value = random.randint( 1, 49 )
        if not current_value in values:
            values.append( current_value )

    print( "\n".join( map( str, values ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 246 - Task 2 - Python Implementation

Another brute force approach. This time I use `count` to simulate the infinity upper bound.

<br/>
<br/>
```python
import sys
from itertools import count

# task implementation
def main( argv ):
    nums = list( map( int, argv ) )
    ko = len( nums ) - 2

    for current_index in range( 2, len( nums ) ):
        for p in count( start = 1 ):
            for q in count( start = 1 ):
                if nums[ current_index ] == ( p * nums[ current_index - 2 ] + q * nums[ current_index - 1 ] ) \
                  or nums[ current_index ] == ( p * -1 * nums[ current_index - 2 ] + q * nums[ current_index - 1 ] ) \
                  or nums[ current_index ] == ( p * -1 * nums[ current_index - 2 ] + q * -1 * nums[ current_index - 1 ] ) \
                  or nums[ current_index ] == ( p * nums[ current_index - 2 ] + q * -1 *  nums[ current_index - 1 ] ):
                    ok = True
                    ko = ko - 1
                    break

            if ok:
                break

    if ko == 0:
        print( 'True' )
    else:
        print( 'False' )



# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )



```
<br/>
<br/>
