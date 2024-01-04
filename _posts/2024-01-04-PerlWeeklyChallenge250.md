---
layout: post
title:  "Perl Weekly Challenge 250: the first one of 2024!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 250: the first one of 2024!

This post presents my solutions to the [Perl Weekly Challenge 250](https://perlweeklychallenge.org/blog/perl-weekly-challenge-250/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 250 - Task 1 - Raku](#task1)
- [PWC 250 - Task 2 - Raku](#task2)
- [PWC 250 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 250 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 250 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 250 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 250 - Task 1 in Python](#task1python)
- [PWC 250 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 250 - Task 1 - Raku Implementation

Given an array of integers, find out the smallest index where `index mod 10 == nums[ index ]`.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    $_.say and exit if ( @nums[ $_ ] == ( $_ % 10 ) ) for 0 ..^ @nums.elems;
    '-1'.say;
}

```
<br/>
<br/>

I loop over all the possible indexes, test the condition and print the index. If I found one, I exit, otherwise the catchall `-1` value is printed.


<a name="task2"></a>
## PWC 250 - Task 2 - Raku Implementation

Given an array of alphanumeric stuff, either numbers or letters, calculate the max string value assuming that:
- for a string the value is its own length;
- for a digit-only string it is its numerical value.

<br/>
<br/>
```raku
sub MAIN( *@words ) {
    @words.map( { $_ ~~ / ^ <[0..9]>+ $ / ?? $_.Int !! $_.chars } ).max.say;
}

```
<br/>
<br/>

This is one of those rare cases where the second task is simpler than the first one: I remap the array of words into their numerical values. If the current word is made only by digits, I convert it into an integer, otherwise I compute its length. Then I keep the `max` value.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 250 - Task 1 - PL/Perl Implementation

A single loop does suffice to find out the smallest index, since I loop from the first one and exit the function as soon as I find one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc250.task1_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $nums ) = @_;

   for  ( 0 .. $nums->@* - 1 ) {
       return $_ if ( ( $_ % 10 ) == $nums->@[ $_ ] );
   }

   return -1;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 250 - Task 2 - PL/Perl Implementation

Similar idea to the Raku approach: if it looks like a number, convert the string into a number, otherwise compute the length.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc250.task2_plperl( text[] )
RETURNS int
AS $CODE$
   my ( $words ) = @_;
   my $max = 0;

   for ( $words->@* ) {
       my $value = 0;

       if ( $_ =~ / ^ \d+ $ /x ) {
       	  $value = int( $_ );
       }
       else {
       	  $value = length( $_ );
       }

       $max = $value if ( $value > $max );

   }

   return $max;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 250 - Task 1 - PL/PgSQL Implementation


Quite short implementation: I loop on every index and check if the modulus is a good value, stopping at the very first index found.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc250.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$
BEGIN
	FOR i IN 1 .. array_length( nums, 1 ) LOOP
	    IF  mod( i, 10 ) = nums[ i ] THEN
	    	RETURN i;
	    END IF;
	END LOOP;

	RETURN -1;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 250 - Task 2 - PL/PgSQL Implementation

Here I decided to use a temproary table to store the word `w`, its length `l` and its numerical value `v`.
At first I insert the word and its length, then I update every *looks-like a number* entry with its value, and then select the max value I find in the table.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc250.task2_plpgsql( words text[] )
RETURNS int
AS $CODE$
DECLARE
	final int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS t_values( w text, l int, v int DEFAULT 0 );
	TRUNCATE t_values;

	INSERT INTO t_values( w, l, v )
	SELECT vv, length( vv ), length( vv )
	FROM unnest( words ) vv;

	UPDATE t_values
	SET v = w::int
	WHERE w IN ( SELECT w FROM t_values
	      	     WHERE regexp_match( w, '^\d+$' ) IS NOT NULL );

	SELECT max( v )
	INTO final
	FROM t_values;

	RETURN final;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 250 - Task 1 - Python Implementation

The approach is the same as in Raku, with the extra work of converting every value into an integer.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    for i in range(0,len(argv)):
        if int(i) % 10 == int( argv[ i ] ):
            return i

    return -1


# invoke the main without the command itself
if __name__ == '__main__':
    print( main( sys.argv[ 1: ] ) )
```
<br/>
<br/>



<a name="task2python"></a>
## PWC 250 - Task 2 - Python Implementation

I use a compiled regex to match what it looks like a number, and in such case I convert it into an integer, otherwise compute the length.

<br/>
<br/>
```python
import sys
import re

# task implementation
def main( argv ):
    max = 0
    is_num = re.compile( '^\d+$' )
    for current in argv:
        v = 0

        if is_num.match( current ) is None:
            v = len( current )
        else:
            v = int( current )

        if v > max:
            max = v

    return max


# invoke the main without the command itself
if __name__ == '__main__':
    print( main( sys.argv[ 1: ] ) )
```
<br/>
<br/>
