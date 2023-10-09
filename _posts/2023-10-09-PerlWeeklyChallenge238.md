---
layout: post
title:  "Perl Weekly Challenge 238: running sums and multiplications (and Python for the very first time!)"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 238: running sums and multiplications (and Python for the very first time!)

This post presents my solutions to the [Perl Weekly Challenge 238](https://perlweeklychallenge.org/blog/perl-weekly-challenge-238/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 238 - Task 1 - Raku](#task1)
- [PWC 238 - Task 2 - Raku](#task2)
- [PWC 238 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 238 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 238 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 238 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 238 - Task 2 (another implementation )in PostgreSQL PL/PgSQL](#task2plpgsql_b)
- [PWC 238 - Task 1 - Python](#task1python)
- [PWC 238 - Task 2 - Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 238 - Task 1 - Raku Implementation

The first task was about computing the *running sum* of an array of integers, defined as the sum of the *up-to-nth element*.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    my @running-sum;
    for 0 ..^ @nums.elems -> $index {
		@running-sum[ $index ] = [+] @nums[ 0 .. $index ];
    }

    @running-sum.join( ', ' ).say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 238 - Task 2 - Raku Implementation

The second task was about sorting an array of integers by the value, assuming the value has to be a single digit. If the current element was more than one digit, the digits have to be multiplied together unless a single digit element is found. Then, all the elements with the same number of multiplication steps have to be sorted together and an higher level than those with less steps.


<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( { $_ ~~ Int && $_ > 0 } ).elems == @nums.elems } ) {
    my %steps;
    for @nums {
		my $step-counter = 0;
		my $value = $_;
		while ( $value > 9 ) {
		    $value = [*] $value.comb;
		    $step-counter++;
		}

		%steps{ $step-counter }.push: $_;
    }

    my @running-sort.push: | %steps{ $_ }.sort for %steps.keys.sort;
    @running-sort.join( ', ' ).say;

}

```
<br/>
<br/>

The idea is to keep an hash, named `%steps` keyed by the number of required steps to reduce the value to a single digit, and then store the current value into the number of steps. Then, it is only a matter of sorting the hash by its keys and the corresponding array by their values.


# PL/Perl Implementations



<a name="task1plperl"></a>
## PWC 238 - Task 1 - PL/Perl Implementation

Same as the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc238.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   for my $index ( 0 .. $nums->@* - 1 ) {
       my $sum = 0;

       $sum += $_ for ( $nums->@[ 0 .. $index ] );
       return_next( $sum );
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 238 - Task 2 - PL/Perl Implementation


Same idea as per the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc238.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   my $steps = {};

   # utility function to reduce a number
   # does only one pass so that I can counter
   # how many passes are required
   my $reduce = sub {
      my ( $number ) = @_;
      return $number if ( $number <= 9 );

      my $value = 1;
      for my $digit ( split( '', $number ) ) {
       	  $value *= $digit;
      }

      return $value;
   };

   for ( $nums->@* ) {
       my $step_counter = 0;
       my $value = $_;

       while ( $value > 9 ) {
       	     $value = $reduce->( $value );
	     $step_counter++;
       }

       push $steps->{ $step_counter }->@*, $_;
   }

   for my $key ( sort keys $steps->%* ) {
       return_next( $_ ) for ( sort $steps->{ $key }->@* )
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


Here I use the `$reduce` utility function to perform a single step of multiplication in the case the number has more than one digit. Performign a single step makes it possible to count how many times a reduction happened.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 238 - Task 1 - PL/PgSQL Implementation

Here I use a *window function* to perform the sum.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc238.task1_plpgsql( nums int[] )
RETURNS TABLE( v int, s int )
AS $CODE$

   SELECT v, sum( v ) OVER ( ORDER BY v )
   FROM unnest( nums ) v;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


The `sum` function is both an aggregate and a window function: when used as a window function it provides the sum of the elements over a defined window, that in our case is just the window order by the value.

<a name="task2plpgsql"></a>
## PWC 238 - Task 2 - PL/PgSQL Implementation

Similar to the Raku approach, but I use a temporary table as an hash to store the values, the multiplications and the steps.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc238.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	current_value int;
	digit text;
	multiplication int;
	step_counter int;
	value_to_insert int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS mul( v int, mul int, steps int DEFAULT 0 );
	TRUNCATE mul;

	FOREACH current_value IN ARRAY nums LOOP
		IF current_value < 9 THEN
		   INSERT INTO mul( v, mul )
		   VALUES( current_value, current_value );
		   CONTINUE;
		END IF;

		-- if here the number is at least two digits long
		step_counter := 0;
		value_to_insert := current_value;
		WHILE current_value > 9 LOOP
		      multiplication := 1;
		      step_counter := step_counter + 1;

		      FOREACH digit IN ARRAY regexp_split_to_array( current_value::text, '' ) LOOP
		      	      multiplication := multiplication * digit::int;
		      END LOOP;

		      current_value := multiplication;
		END LOOP;

		INSERT INTO mul( v, mul, steps )
		VALUES( value_to_insert, multiplication, step_counter );

	END LOOP;


	RETURN QUERY SELECT v
	       	     FROM mul
		     ORDER BY steps ASC, v ASC;
END

$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


Note how verbose it does result this solution, when compared to PL/Perl and Raku ones. This again emphasizes how PL/PgSQL is probably not the best suited language for number mangling and non-set operations.




<a name="task2plpgsql_b"></a>
## PWC 238 - Task 2 - Another PL/PgSQL Implementation

Defining a function to *reduce* a single value, it is possible to quickly come up with a single query solution.
First of all, the function to reduce a value to a single digit returning the number of steps is:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION pwc238.reduce( n int )
RETURNS int
AS $CODE$
DECLARE
	current_value int;
	step_counter  int;
	digit text;
	multiplication int;

BEGIN
	current_value := n;
	step_counter  := 0;

	WHILE current_value > 9 LOOP
	      multiplication := 1;
	      step_counter   := step_counter + 1;

	      FOREACH digit IN ARRAY regexp_split_to_array( current_value::text, '' ) LOOP
	      	      multiplication := multiplication * digit::int;
	      END LOOP;

	      current_value := multiplication;
	END LOOP;

	RETURN step_counter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


Then, the function to use a single query to get the result is as simple as ordering values by the result of the above function:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc238.task2_plpgsql( nums int[] )
RETURNS SETOF INT
AS $CODE$

SELECT v
FROM unnest( nums ) v
ORDER BY pwc238.reduce( v ), v;

$CODE$
LANGUAGE sql
VOLATILE
;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 238 - Task 1 - Python Implementation

Baby steps in my Python knowledge.

<br/>
<br/>
```python
import sys

def main( argv ):
    running_sum = []
    current_index = 0
    while current_index < len( argv ):
        running_sum.insert( current_index, 0 )

        for n in argv[ 0 : current_index   ]:
            running_sum[ current_index ] += int( n )

        current_index += 1

    print( ", ".join( map( str, running_sum ) ) )




if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>


The `main` function receives the expected array of numbers, and does the computation of the running sum as for the Raku and PL/Perl implementation.
Note how horrible it is to having to `map` strings out of integers due to the need for `join` to use only strings.

<a name="task2python"></a>
## PWC 238 - Task 2 - Python Implementation

The second task was much more verbose than the other languages.

<br/>
<br/>
```python
import sys
import collections

def reduce( n ):
    if n <= 9:
        return n

    multiplication = 1
    for digit in map( int, str( n ) ):
        multiplication *= digit

    return multiplication


def main( argv ):
    steps = {}
    for n in map( int, argv ):
        current_step = 0
        if n > 9:
            # need reduction
            current_value = n
            while current_value > 9:
                current_value = reduce( current_value )
                current_step += 1

        # if the key is not here, create a list
        if not str( current_step )  in steps:
            steps[ str( current_step ) ] = []

        steps[ str(current_step) ].append( n )

    # now traverse the dictionary and sort the array
    # and print it
    for k, v in collections.OrderedDict(sorted(steps.items())).items():
        print( ", ".join( map( str, v ) ) )



if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>

The `reduce` function does a single pass in reducing a more than one digit number.
The `main` function does all the job. Please note how hard it is placing a list into a dictionary item, seems Python has nothing like *autovifification*.
