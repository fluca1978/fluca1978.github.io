---
layout: post
title:  "Perl Weekly Challenge 348: splitting, counting and multiplying"
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

# Perl Weekly Challenge 348: splitting, counting and multiplying

This post presents my solutions to the [Perl Weekly Challenge 348](https://perlweeklychallenge.org/blog/perl-weekly-challenge-348/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 348 - Task 1 - Raku](#task1)
- [PWC 348 - Task 2 - Raku](#task2)
- [PWC 348 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 348 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 348 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 348 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 348 - Task 1 in PL/Java](#task1pljava)
- [PWC 348 - Task 2 in PL/Java](#task2pljava)
- [PWC 348 - Task 1 in Python](#task1python)
- [PWC 348 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 348 - Task 1 - Raku Implementation

The first task was about to find out if a given string can be split into two equally length parts, left and right, with the same amount of vowels.

<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    # if the string has an even length, nothing to do
    'False'.say and exit( 1 ) if ( $string.chars !%% 2 );
    'False'.say and exit( 2 ) unless ( $string.comb[ 0 ..^ $string.chars / 2 ].fc.grep( * ~~ /<[aeiou]>/ ).elems ==
				       $string.comb[ $string.chars / 2  .. * ].fc.grep( * ~~ /<[aeiou]>/ ).elems );


    'True'.say;
}

```
<br/>
<br/>

The idea is to check if the length of the string is odd, in such case the program can immediatly terminate.
Then I split the string into two arrays of folded-case chars, that in turn are passed into `grep` to extract the voewls. If the count of the elements of the arrays are the same, then success, otherwise failure.



<a name="task2"></a>
## PWC 348 - Task 2 - Raku Implementation


Given two string that represent a time format `'HH::MM'`, and given a list of possible time adjustments in minutes `60, 15, 10, 5, 1`, compute how many operations are required to move from the first time to the second.

<br/>
<br/>
```raku
sub MAIN( Str $source, Str $target where { $source ~~ / ^ \d ** 2 <[:]> \d ** 2 $ / && $target ~~ / ^ \d ** 2 <[:]> \d ** 2 $ / } ) {
    my %begin = hour => $source.split( ':' )[ 0 ],
		mins => $source.split( ':' )[ 1 ];

    my %end = hour => $target.split( ':' )[ 0 ],
	      mins => $target.split( ':' )[ 1 ];


    %begin<mins> = %begin<mins> + %begin<hour> * 60;
    %end<mins>   = %end<mins> + %end<hour> * 60;

    my @operations;

    for ( 60, 15, 10, 5, 1 ) -> $current {
		while ( %end<mins> - %begin<mins> >= $current ) {
		    %begin<mins> += $current;
		    @operations.push: $current;

		}
    }

    @operations.elems.say;

}

```
<br/>
<br/>


The solution is probably too much complicated, however I first compute the `hour` and `mins` part of each string, and place it into an hash. Then compute only the minutes of every time specified.
The trick now is to iterate on the available adjustement, from the biggest to the smallest, and see if such operation can fit.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 348 - Task 1 - PL/Perl Implementation

Same idea as in Raku, simply more verbose.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc348.task1_plperl( text )
RETURNS boolean
AS $CODE$

   my ( $text ) = @_;

   return 0 unless ( length( $text ) % 2 == 0 );

   my @chars = split //, $text;
   my ( @left, @right ) = ( @chars[ 0 .. $#chars / 2 - 1 ],
                            @chars[ $#chars / 2 .. $#chars ] );

  return scalar( grep( { $_ =~ / [aeiou] / } @left ) ) == scalar( grep( { $_ =~ / [aeiou] / } @right ) );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 348 - Task 2 - PL/Perl Implementation

Same as in Raku: compute the time in minutes, and then add every possible operation from the biggest to the smallest.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc348.task2_plperl( text, text )
RETURNS int
AS $CODE$

   my ( $source, $target ) = @_;

   my @elems = split /[:]/, $source;
   my $src_mins = $elems[ 0 ] * 60 + $elems[ 1 ];

   @elems = split /[:]/, $target;
   my $dst_mins = $elems[ 0 ] * 60 + $elems[ 1 ];

   my @operations;
   for my $current ( 60, 15, 10, 5, 1 ) {
       while ( $dst_mins - $src_mins >= $current ) {
       	     push @operations, $current;
	     $src_mins += $current;
       }
   }

   return scalar( @operations );


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 348 - Task 1 - PL/PgSQL Implementation

I use regular expressions to split a string into its chars and count the number of vowels.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc348.task1_plpgsql( s text )
RETURNS boolean
AS $CODE$
DECLARE
	lc int;
	rc int;
	sz int;
BEGIN
	if mod( length( s ), 2 ) <> 0 then
	   return false;
	else
	  sz := length( s ) / 2;
	end if;

	SELECT count( t.* )
	INTO lc
	FROM ( SELECT regexp_matches( lower( left( s, sz ) ), '[aeiou]' ) )  t;

	SELECT count( t.* )
	INTO rc
	FROM ( SELECT regexp_matches( lower( right( s, sz ) ), '[aeiou]' ) )  t;


	IF lc = rc THEN
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
## PWC 348 - Task 2 - PL/PgSQL Implementation

Again, the trick is to compute the times into minutes and compare iterating over the operations.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc348.task2_plpgsql( s text, t text )
RETURNS int
AS $CODE$
DECLARE
	operations int := 0;
	src_mins   int := 0;
	dst_mins   int := 0;
	multiplier int := 0;
	v          text;
	x          int;
BEGIN

	multiplier := 60;
	FOR v IN SELECT * FROM regexp_split_to_table( s, '[:]' ) LOOP
	    src_mins := src_mins + v::int * multiplier;
	    multiplier := 1;
	END LOOP;

	multiplier := 60;
	FOR v IN SELECT * FROM regexp_split_to_table( t, '[:]' ) LOOP
	    dst_mins := dst_mins + v::int * multiplier;
	    multiplier := 1;
	END LOOP;

	FOREACH x IN ARRAY array[ 60, 15, 10, 5, 1 ]::int[] LOOP
		WHILE dst_mins - src_mins >= x  LOOP
		      src_mins := src_mins + x;
		      operations := operations + 1;
		END LOOP;
	END LOOP;

	RETURN operations;


END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 348 - Task 1 - PostgreSQL PL/Java Implementation

Same as in the previous implementations.

<br/>
<br/>
```java
    @Function( schema = "pwc348",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static boolean task1_pljava( String text ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc348.task1_pljava" );

		if ( text == null || text.isEmpty() || text.length() %2 != 0 )
		    return false;

		int left = 0, right = 0, middle = text.length() / 2;
		int index = 0;
		String vowels = "aeiou";
		for ( String c : text.split( "" ) ) {
		    index++;

		    if ( vowels.indexOf( c ) < 0 )
				continue;

		    if ( index <= middle )
				left++;
		    else
				right++;
		}

		return left == right;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 348 - Task 2 - PostgreSQL PL/Java Implementation


Same as in the previous implementations.

<br/>
<br/>
```java
    public static final int task2_pljava( String source, String target ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc348.task2_pljava" );

		int src_mins = 0;
		int dst_mins = 0;
		int[] multipliers = { 60, 1 };
		List<Integer> operations = new LinkedList<Integer>();

		int index = 0;
		for ( String part : source.split( ":" ) )
		    src_mins += Integer.parseInt( part ) * multipliers[ index++ ];


		index = 0;
		for ( String part : target.split( ":" ) )
		    dst_mins += Integer.parseInt( part ) * multipliers[ index++ ];


		int ops[] = { 60, 15, 10, 5, 1 };
		for ( int op : ops )
		    while ( dst_mins - src_mins >= op ) {
				src_mins += op;
				operations.add( op );
		    }


		return operations.size();

    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 348 - Task 1 - Python Implementation

The basic idea is the same as in the previous implementations, but here I can iterate directly over the string chars.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    lc     = 0
    rc     = 0
    middle = len( args[ 0 ] ) / 2
    index  = 0
    vowels = [ 'a', 'e', 'i', 'o', 'u' ]

    for c in args[ 0 ].lower() :
        index += 1

        if c in vowels :
            if index <= middle :
                lc += 1
            else :
                rc += 1

    return lc == rc



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 348 - Task 2 - Python Implementation

Same implementation as in PL/Java.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    source = args[ 0 ]
    target = args[ 1 ]
    multi  = [ 60, 1 ]
    index  = 0

    src_mins = 0
    dst_mins = 0


    for p in source.split( ':' ) :
        src_mins += int( p ) * multi[ index ]
        index    += 1

    index = 0
    for p in target.split( ':' ) :
        dst_mins += int( p ) * multi[ index ]
        index    += 1

    op_list = []
    operations = [ 60, 15, 10, 5, 1 ]
    for op in operations :
        while dst_mins - src_mins >= op :
            src_mins += op
            op_list.append( op )

    return len( op_list )

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
