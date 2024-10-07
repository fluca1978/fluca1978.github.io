---
layout: post
title:  "Perl Weekly Challenge 290: arrays and numbers"
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

# Perl Weekly Challenge 290: arrays and numbers

This post presents my solutions to the [Perl Weekly Challenge 290](https://perlweeklychallenge.org/blog/perl-weekly-challenge-290/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 290 - Task 1 - Raku](#task1)
- [PWC 290 - Task 2 - Raku](#task2)
- [PWC 290 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 290 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 290 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 290 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 290 - Task 1 in PL/Java](#task1pljava)
- [PWC 290 - Task 2 in PL/Java](#task2pljava)
- [PWC 290 - Task 1 in Python](#task1python)
- [PWC 290 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 290 - Task 1 - Raku Implementation

The first task was, given an array of integers, to find out if there are two different indexes so that the element of one index is half the element of the other index. I decided to quickly implement with a nested loop.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.elems == @numbers.grep( * ~~ Int ).elems } ) {
    my @found;

    for 0 ..^ @numbers.elems -> $i {
		for $i ..^ @numbers.elems -> $j {
			@found.push: [ $i, $j ] if ( @numbers[ $i ] == 2 * @numbers[ $j ] );
		}
    }

    'True'.say and exit if ( @found );
    'False'.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 290 - Task 2 - Raku Implementation

The second task was to analyze a string, mainly made by digits (but not only), so that the last digit can be a checksum according to some rules.
The sum of the digits (except) the checksum must be a number so that adding the checksum the final result is modulo `10`. When summing all the digits, if a single digit has to be multiplied by `2`, and if the result is a two digits, sum the digits by their own.
Moreover, two digits with the same value and adiacent positions have not be considered twice.

<br/>
<br/>
```raku
sub MAIN( $digits ) {

    my @numbers = $digits.comb( :skip-empty ).grep( * ~~ /\d/ );
    my $payload = @numbers[ * - 1 ];

    my @checksums;
    my $last;
    for 2 .. @numbers.elems {
		next if ( $last && $last == @numbers[ * - $_ ] );

	    my $current = @numbers[ * - $_ ] * 2;
		$current = $current.comb( :skip-empty ).map( *.Int ).sum if ( $current > 9 );
		@checksums.push: $current;
		$last = @numbers[ * - $_ ];
    }


    say 'True' and exit if ( ( @checksums.sum + $payload ) %% 10 );
    say 'False';
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 290 - Task 1 - PL/Perl Implementation

Similar implementation to the Raku one, but a little more verbose. I use an inner `$sum` function to sum the digits of a two digits string.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc290.task1_plperl( int[] )
RETURNS bool
AS $CODE$

   my ( $numbers ) = @_;
   my @found;

   for my $i ( 0 .. $numbers->@* - 1 ) {
       for my $j ( $i + 1 .. $numbers->@* - 1 ) {
       	   push @found, [ $i, $j ] if ( $numbers->[ $i ] == 2 * $numbers->[ $j ] );
       }
   }

   return @found > 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 290 - Task 2 - PL/Perl Implementation

Same implementation as in Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc290.task2_plperl( text )
RETURNS bool
AS $CODE$

   my ( $string ) = @_;
   my @numbers = grep { $_ =~ /\d/ } split //, $string;

   my $sum = sub {
      my $value = 0;
      for my $v ( split //, $_[ 0 ] ) {
      	  $value += $v;
      }

      return $value;
   };

   my $payload = $numbers[ -1 ];
   # my @checksums = map { $_ <= 9 ? $_ : $sum->( $_ ) }
   #                 map { $_ * 2 }
   # 		   @numbers[ 0 .. $#numbers - 1 ];


   my @checksums;
   my $last;
   for my $v ( @numbers[ 0 .. $#numbers - 1 ] ) {
       next if ( $last && $last == $v );

       $v *= 2;
       if ( $v > 9 ) {
       	  $v = $sum->( $v );
       }

       push @checksums, $v;
   }

   my $check = 0;
   for ( @checksums ) {
       $check += $_;
   }


   $check += $payload;
   return 1 if ( $check % 10 == 0 );
   return 0;




$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 290 - Task 1 - PL/PgSQL Implementation

Quite simple implementation using nested loops.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc290.task1_plpgsql( nums int[] )
RETURNS boolean
AS $CODE$
DECLARE
	i int;
	j int;
	c int := 0;
BEGIN

	FOR i IN 1 .. array_length( nums, 1 ) - 1 LOOP
	    FOR j IN ( i + 1 ) .. array_length( nums, 1 ) LOOP
	    	IF nums[ i ] = ( nums[ j ] * 2 ) THEN
		   c := c + 1;
		END IF;
	    END LOOP;
	END LOOP;

	IF c > 0 THEN
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



<a name="task2plpgsql"></a>
## PWC 290 - Task 2 - PL/PgSQL Implementation

Essentially, the same implementation as in Raku, but using queries to compute sums.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc290.task2_plpgsql( s text )
RETURNS boolean
AS $CODE$
DECLARE
	payload  int;
	checksum int := 0;
	n int;
	last_seen int;
BEGIN
	SELECT right( s, 1 )
	INTO payload;

	FOR n IN SELECT v::int FROM regexp_split_to_table( s, '' ) v LIMIT length( s ) - 1 LOOP
	    IF last_seen IS NOT NULL AND last_seen = n THEN
	       CONTINUE
	    END IF;

	    n := n * 2;
	    IF n > 9 THEN
	       SELECT sum( v::int )
	       INTO n
	       FROM regexp_split_to_table( n::text, '' ) v;
	    END IF;

	    checksum := checksum + n;
	    last_seen = n;
	END LOOP;

	checksum := checksum + payload;
	IF mod( checksum, 10 ) = 0 THEN
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
## PWC 290 - Task 1 - PostgreSQL PL/Java Implementation

Again, nested loops to the rescue.

<br/>
<br/>
```java
    public static final boolean task1_pljava( int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc290.task1_pljava" );

		int counter = 0;

		for ( int i = 0; i < numbers.length - 1; i++ )
		    for ( int j = 0; j < numbers.length; j++ )
			if ( numbers[ i ] == numbers[ j ] * 2 )
			    counter++;

		return counter > 0;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 290 - Task 2 - PostgreSQL PL/Java Implementation

Same implementation as in PL/Perl, but longer.


<br/>
<br/>
```java
    public static final boolean task2_pljava( String s ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc290.task2_pljava" );

		List<Integer> numbers = new LinkedList<Integer>();

		for ( String v : s.split( "" ) ) {
		    try {
				int i = Integer.parseInt( v );
				numbers.add( i );
		    } catch ( Exception e ) { }
		}

		int payload = numbers.get( numbers.size() - 1 );
		int checksum = 0;
		Integer last = null;

		for ( int i = 0; i < numbers.size() - 1; i++ ) {
			if ( last != null && last == numbers.get( i ) )
				continue;
				
		    int current = numbers.get( i ) * 2;
		    if ( current > 9 ) {
				int sum = 0;
				for ( String v : ( "" + current ).split( "" ) )
			    sum += Integer.parseInt( v );
				current = sum;
		    }

		    checksum += current;
		}

		return ( checksum + payload ) % 10 == 0;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 290 - Task 1 - Python Implementation

Following the idea of the other implementations, the code looks like the other examples.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    counter = 0
    args = list( map( int, args ) )

    for i in range( 0, len( args ) ):
        for j in range( i + 1, len( args ) ):
            if args[ i ] == ( 2 * args[ j ] ):
                counter +=1

    return counter > 0

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 290 - Task 2 - Python Implementation

Similar implementation, even if there could be a clash in already seen values since I map the values to their double before comparing if they have been already seen. Therefore `5` and `14` will be managed as already seen, since this is not clear from the specification of the task, I assume this little diference is acceptable.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    numbers = list( map( int, args ) )
    payload = numbers[ -1 ]
    checksum = 0

    numbers = list( map( lambda x: x * 2, numbers[ 0 : -1 ] ) )
	last_seen = Nil
    for v in numbers:
		if last_seen is not Nil and last_seen == v:
			continue
			
        if v > 9:
            s = 0
            for x in str( v ):
                s += int( x )
            v = s

        checksum += v
		last_seen = v

    if ( checksum + payload ) % 10 == 0:
        return True
    else:
        return False



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
