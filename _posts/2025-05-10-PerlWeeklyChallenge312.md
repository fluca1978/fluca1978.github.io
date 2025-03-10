---
layout: post
title:  "Perl Weekly Challenge 312: it's raining letters!"
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

# Perl Weekly Challenge 312: it's raining letters!

This post presents my solutions to the [Perl Weekly Challenge 312](https://perlweeklychallenge.org/blog/perl-weekly-challenge-312/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 312 - Task 1 - Raku](#task1)
- [PWC 312 - Task 2 - Raku](#task2)
- [PWC 312 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 312 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 312 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 312 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 312 - Task 1 in PL/Java](#task1pljava)
- [PWC 312 - Task 2 in PL/Java](#task2pljava)
- [PWC 312 - Task 1 in Python](#task1python)
- [PWC 312 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 312 - Task 1 - Raku Implementation

The first task was about computing how many seconds a typewriter used to print a set of given letters, assuming there is one second spent for printing a char and another second to move the rotating letter of one step.

<br/>
<br/>
```raku
sub MAIN( Str $typing ) {
    my $secs = 0;
    my $position = 0;
    my @letters = 'a' .. 'z';

    for $typing.lc.comb -> $current_letter {
		my $next_position = @letters.grep( * ~~ $current_letter, :k ).first().Int;
		my $clockwise = ( $next_position - $position ) % @letters.elems;
		my $anti_clockwise = abs( @letters.elems - $next_position + 1 );
		$position = $next_position;
		$secs += 1 + ( $clockwise, $anti_clockwise ).min;
    }

    $secs.say;
}

```
<br/>
<br/>

The problem of the task is that we can rotate the letters in a clockwise and anti-clockwise direction, so we need to find out the shortest path to every letter.




<a name="task2"></a>
## PWC 312 - Task 2 - Raku Implementation

The second task, given a string of one letter and number repeated, where each letter represent an `RGB` color and the number the label of a box, the aim was to find out the boxes containing all the colors.


<br/>
<br/>
```raku
sub MAIN( Str $elements ) {

    my %boxes;
    for $elements.uc.comb -> $color, $box {
		%boxes{ $box }.push: $color;
    }

    my $found = 0;
    $found++ if %boxes{ $_ }.values.grep( * ~~ /R|G|B/ ).elems >= 3 for %boxes.keys;
    $found.say;
}

```
<br/>
<br/>


I use an hash to store, for every `$box`, the list of `$color` found in the string. Then, I `grep` the values of the `%boxes` to see how many `R` and `G` and `B` I can find in  a single box, and if that is more than `3` the box contains all the colors.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 312 - Task 1 - PL/Perl Implementation

Same implementation as in Raku, a little more verbose.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc312.task1_plperl( text )
RETURNS int
AS $CODE$

   my ( $typing ) = @_;
   my %letters;
   my $position = 0;
   my $count = 0;

   $letters{ $_ } = $count++ for ( 'a' .. 'z' );
   $count = 0;

   for my $current_letter ( split //, lc( $typing ) ) {
       my $next_position = $letters{ $current_letter };
       my ( $clock, $anti_clock ) = ( $next_position - $position, 25 - $next_position - $position + 1 );
       $clock *= $clock < 0 ? -1 : 1;
       $anti_clock *= $anti_clock < 0 ? -1 : 1;
       $position = $next_position;
       $count += 1 + ( $clock < $anti_clock ? $clock : $anti_clock );
   }

   return $count;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 312 - Task 2 - PL/Perl Implementation


Same approach as in Raku, with a more verbosity implementation.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc312.task2_plperl( text )
RETURNS int
AS $CODE$

   my ( $boxes ) = @_;

   my @data = split //, uc( $boxes );
   my $result = {};
   my $full = 0;

   while ( @data ) {
   	 my ( $box, $color ) = ( pop @data, pop @data );
	 push $result->{ $box }->@*, $color;
   }

   for my $colors ( values $result->%* ) {
       $full++ if ( scalar( grep { $_ =~ /G|B|R/ } $colors->@* ) >= 3 );
   }

   return $full;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 312 - Task 1 - PL/PgSQL Implementation

I use a temporary table to store every letter with its index when moving clockwise or anticlockwise.
Then, I iterate over each letter in the string and compute the lowest value to move towards.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc312.task1_plpgsql( typing text )
RETURNS int
AS $CODE$
DECLARE
	secs int := 0;
	position int := 1;
	next_position int := 1;
	l int;
	r int;
	letter char;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS letters( c char, clock int, anti int );
	TRUNCATE letters;
	INSERT INTO letters( c, clock, anti )
	SELECT chr( v ), row_number() over (), 25 + 1 - row_number() over ()
	FROM generate_series( 97, 97 + 25 ) v;


	FOR letter IN SELECT v FROM regexp_split_to_table( typing, '' ) v LOOP
	    SELECT abs( clock - position ), abs( 26 - clock - anti - position ), clock
	    INTO l, r, next_position
	    FROM letters
	    WHERE c = letter;


	    IF l > r THEN
	       secs := secs + r;
	    ELSE
	      secs := secs + l;
	    END IF;

	    secs := secs + 1;

	    position := next_position;

	END LOOP;


	RETURN secs;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 312 - Task 2 - PL/PgSQL Implementation


I use a temporary table that, given a box label (number) has the flags to indicate if at least one of the three colors has been found.
I iterate over all the values in the string, and `INSERT` or `UPDATE` the row in the table, so that in the end I can select how many tuples have all the flags set.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc312.task2_plpgsql( boxes text )
RETURNS int
AS $CODE$
DECLARE
	current_color char;
	current_box   int;
	parts         text[];
	result        int;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS boxes( n int, r bool, g bool, b bool, PRIMARY KEY( n ) );
	TRUNCATE boxes;

	SELECT regexp_split_to_array( boxes, '' )
	INTO parts;

	FOR i IN 1 .. length( boxes ) / 2 LOOP
	    current_color := parts[ i + ( i - 1 )  ];
	    current_box   := parts[ i + 1 + ( i - 1 ) ];

	    PERFORM n
	    FROM boxes
	    WHERE n = current_box;

	    IF NOT FOUND THEN
	       INSERT INTO boxes
	       SELECT current_box
	    	   , case current_color when 'R' then true else false end
		   , case current_color when 'G' then true else false end
		   , case current_color when 'B' then true else false end
	      ;
	   ELSE
		UPDATE boxes
		SET
			  r = r OR case current_color when 'R' then true else false end
			, g = g OR case current_color when 'G' then true else false end
			, b = b OR case current_color when 'B' then true else false end
		WHERE n = current_box;

	   END IF;


	END LOOP;

	SELECT count(*)
	INTO result
	FROM boxes
	WHERE r AND g and b;

	RETURN result;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 312 - Task 1 - PostgreSQL PL/Java Implementation

Same approach as in the other implementations.


<br/>
<br/>
```java
    public static final int task1_pljava( String typing ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc312.task1_pljava" );

		String alphabet = "abcdefghijklmnopqrstuvwxyz";
		Map<String,Integer> letters = new HashMap<String, Integer>();
		int position = 0;
		int secs = 0;
		int next_position = 0;

		for ( String c : alphabet.split( "" ) )
		    letters.put( c, secs++ );

		secs = 0;

		for ( String current_letter : typing.toLowerCase().split( "" ) ) {
		    next_position = letters.get( current_letter );
		    int clock = Math.abs( next_position - position );
		    int anti = Math.abs( alphabet.length() - next_position - position + 1 );

		    secs += 1 + ( clock > anti ? anti : clock );
		    position = next_position;
		}

		return secs;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 312 - Task 2 - PostgreSQL PL/Java Implementation


Same implementation as in the other languages.


<br/>
<br/>
```java
    public static final int task2_pljava( String boxes ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc312.task2_pljava" );

		List<String> data = new LinkedList<String>();
		for ( String c :  boxes.toUpperCase().split( "" ) )
		    data.add( c );

		Map<String, String> found = new HashMap<String, String>();


		while ( data.size() > 0 ) {
		    String color = data.remove( 0 );
		    String box   = data.remove( 0 );


		    found.putIfAbsent( box, "" );
		    found.put( box, found.get( box ) + color );
		}

		int ok = 0;
		for ( String box : found.keySet() ) {
		    if ( found.get( box ).contains( "R" )
			 && found.get( box ).contains( "G" )
			 && found.get( box ).contains( "B" ) )
			ok++;
		}

		return ok;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 312 - Task 1 - Python Implementation

The idea is similar to that of the other language implementations.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    alphabet = 'abcdefghijklmnopqrstuvwxyz'
    typing = args[ 0 ]
    letters = {}
    secs = 0
    position = 0
    next_position = 0

    for x in alphabet :
        letters[ x ] = position
        position += 1

    for c in typing :
        next_position = letters[ c ]
        clock = abs( next_position - position )
        anti  = abs( len( alphabet ) - next_position - position )
        position = next_position

        secs += 1
        if clock > anti :
            secs += anti
        else :
            secs += clock

    return secs


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 312 - Task 2 - Python Implementation


Similar to other implementations before.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    boxes = {}
    data  = args[ 0 ]

    while len( data ) > 0 :
        color, box, data = data[ 0 ], data[ 1 ], data[ 2: ]
        if not box in boxes :
            boxes[ box ] = []

        boxes[ box ].append( color )

    found = 0
    for box in boxes :
        if "R" in boxes[ box ] and "G" in boxes[ box ] and "B" in boxes[ box ]:
            found += 1

    return found



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
