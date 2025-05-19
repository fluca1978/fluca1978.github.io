---
layout: post
title:  "Perl Weekly Challenge 322: Splitting and Sorting"
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

# Perl Weekly Challenge 322: Splitting and Sorting

This post presents my solutions to the [Perl Weekly Challenge 322](https://perlweeklychallenge.org/blog/perl-weekly-challenge-322/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 322 - Task 1 - Raku](#task1)
- [PWC 322 - Task 2 - Raku](#task2)
- [PWC 322 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 322 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 322 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 322 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 322 - Task 1 in PL/Java](#task1pljava)
- [PWC 322 - Task 2 in PL/Java](#task2pljava)
- [PWC 322 - Task 1 in Python](#task1python)
- [PWC 322 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 322 - Task 1 - Raku Implementation

The first task was about splitting a string into a dashed one, assuming it is already coming as a dashed-splitted string, but reorganizing letters into groups of the given size.

<br/>
<br/>
```raku
sub MAIN( Str $string, Int $size where { $size <= $string.chars } ) {
    $string.split( '-' ).join.flip.comb( :skip-empty ).rotor( $size, :partial ).map( *.join ).join( '-' ).flip.say;
}

```
<br/>
<br/>


A single line does suffice: I `split` the string by the dashes and recombine it via `join`, then separate into an array of chars by means of `comb`, then `rotor` by the size (assuming I've reversed the string, since only the first group could be shorter than the size), then recombine the single pieces into a dashed (flipped again) string.


<a name="task2"></a>
## PWC 322 - Task 2 - Raku Implementation


The second task was about providing the position of a number in a given array when sorted.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems } ) {
    my @result;
    for @numbers -> $current {
		@result.push: @numbers.sort.grep( * ~~ $current, :k ).first + 1;
    }

    @result.say;
}

```
<br/>
<br/>


I iterate over every element of the array, and extract the `k`ey from the sorted array, taking the first element and pushing into `@result`.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 322 - Task 1 - PL/Perl Implementation

Basically it is the same of the Raku implementation, not having the `rotor` function.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc322.task1_plperl( text, int)
RETURNS text
AS $CODE$

   my ( $string, $size ) = @_;
   die "Size should be less then the length of the string" if ( length( $string ) < $size );

   my @chars = split //, reverse join( '', split( '-', $string ) );
   my $string;
   while ( @chars ) {
   	 $string .= shift @chars for 1 .. $size;
	 $string .= '-' if ( @chars );
   }

   return reverse $string;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 322 - Task 2 - PL/Perl Implementation


Similar to the Raku implementation.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc322.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $numbers ) = @_;

   my @result;

   for my $current ( $numbers->@* ) {
       my $index = 1;
       for ( sort { $a <=> $b } $numbers->@* ) {
       	   if ( $_ == $current ) {
	      push @result, $index;
	      last;
	   }

	   $index++;
       }
   }

   return [ @result ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 322 - Task 1 - PL/PgSQL Implementation

A single query does the trick here.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc322.task1_plpgsql( s text, l int )
RETURNS text
AS $CODE$
DECLARE
	q_match  text;
	q_result text;
BEGIN
	q_match := '(\w{' || l || '})';

	q_match := $$ SELECT reverse( regexp_replace( reverse( regexp_replace( '$$ || s || $$', '-', '', 'g' ) ), '$$ || q_match || $$', '\1-', 'g' ) )$$;

	raise info '[%]', q_match;
	EXECUTE q_match
	INTO q_result;


	RETURN q_result;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I use a regular expression to do the trick of reorganizing the letters in groups of the given size.



<a name="task2plpgsql"></a>
## PWC 322 - Task 2 - PL/PgSQL Implementation

Even here, using a query can do the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc322.task2_plpgsql( n int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
   current int;
   r       int;
BEGIN

	FOR current IN SELECT x FROM unnest( n ) x LOOP
	    WITH sorting AS (
	    	 SELECT row_number() over ( order by y ) as sorted, y
			 FROM   unnest( n ) y
	    )
	    SELECT sorted
	    INTO r
	    FROM sorting
	    WHERE y = current;

	    RETURN NEXT r;

	END LOOP;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The `sorting` CTE provides the array with the position of every element. Then I select the `current` element and get its ordering.



# Java Implementations

<a name="task1pljava"></a>
## PWC 322 - Task 1 - PostgreSQL PL/Java Implementation

A bovine implementation.


<br/>
<br/>
```java
    public static final String task1_pljava( String string, int size ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc322.task1_pljava" );

		List<String> pieces = new LinkedList<String>();

		for ( String s : string.split( "-" ) ) {
		    for ( String q : s.split( "" ) )
				pieces.add( q );
		}

		String result = "";

		for ( int i = 0; i < pieces.size(); ) {
		    for ( int j = 0; j < size; j++ ) {
				if ( i >= pieces.size() )
					break;

	            result += pieces.get( i++ );
		    }

		    if ( i < pieces.size() )
				result += "-";
		}

		return result;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 322 - Task 2 - PostgreSQL PL/Java Implementation

An approach similar to the PL/PgSQL one.

<br/>
<br/>
```java
    public static int[] task2_pljava( int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc322.task2_pljava" );

		List<Integer> sorted = new LinkedList<Integer>();

		for ( int i : numbers )
		    sorted.add( i );

		Collections.sort( sorted );

		int index = 1;
		int c = 0;
		int[] result = new int[ numbers.length ];

		for ( int current : numbers ) {
		    index = 1;

		    for ( int seeking : sorted ) {
				if ( seeking == current )
				    break;
				else
				    index++;
		    }

		    result[ c ] = index;
		    c++;
		}

		return result;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 322 - Task 1 - Python Implementation

Same approach as in PL/Perl.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    string = args[ 0 ]
    size   = int( args[ 1 ] )
    chars  = []
    result = ""

    for c in string[::-1] :
        if c == '-':
            continue

        chars.append( c )

    counter = 1
    for c in chars:
        result  += c
        counter += 1

        if counter % size == 0:
            result += '-'

    return result[::-1]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 322 - Task 2 - Python Implementation

A bovine approach to sort the elements and get the position.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    numbers = list( map( int, args ) )
    sorted_numbers = list( map( int, args ) )
    sorted_numbers.sort()
    result = []

    for c in numbers:
        index = 1
        for x in sorted_numbers:
            if x == c:
                result.append( index )
            else:
                index += 1


    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
