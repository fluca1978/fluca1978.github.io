---
layout: post
title:  "Perl Weekly Challenge 329: mangling strings"
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

# Perl Weekly Challenge 329: mangling strings

This post presents my solutions to the [Perl Weekly Challenge 329](https://perlweeklychallenge.org/blog/perl-weekly-challenge-329/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 329 - Task 1 - Raku](#task1)
- [PWC 329 - Task 2 - Raku](#task2)
- [PWC 329 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 329 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 329 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 329 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 329 - Task 1 in PL/Java](#task1pljava)
- [PWC 329 - Task 2 in PL/Java](#task2pljava)
- [PWC 329 - Task 1 in Python](#task1python)
- [PWC 329 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 329 - Task 1 - Raku Implementation

The first task was about finding out all unique numbers hidden within a give string.

<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / <[a..zA..Z0..9]>+ / } ) {
    my %digits;
    for ( $string ~~ m:g/ <[0..9]>+ / ).values {
		%digits{ $_ }++;
    }

    %digits.keys.sort.join( ',' ).say;
}

```
<br/>
<br/>

I use an hash `%digits` to keep unique values, and a regular expression to match every digit only substring. At last, I print out all the numbers sorted.


<a name="task2"></a>
## PWC 329 - Task 2 - Raku Implementation

The second task was about to find out the longest substring that begins and ends with the same letter but with different case.


<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / <[a..zA..Z]>+ / } ) {
    exit( 0 ) if $string.chars <= 1;

    my @chars = $string.comb;
    my $found;

    for 0 ..^ @chars.elems {
		my $current = @chars[ $_ ];
		my $needle  = $current.uc;
		$needle = $current.lc if ( $current ~~ / <[A..Z]> / );

		my $last-index = @chars[ $_ .. * - 1 ].grep( * ~~ $needle, :k ).max;
		my $match = @chars[ $_ .. $last-index ].join;
		$found = $match if ( ! $found || $match.chars > $found.chars );
    }

    $found.say;

}

```
<br/>
<br/>


I iterate over all the `@chars` of the given string, extracting the `$current` and the searched opposite case one `$needle`. Then I use `grep` to extract the index of the last `$needle`, and then rebuild the substring by slicing and joining the string. If the resulting string is longer than the previously one `$found` I memorize it, and then I print.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 329 - Task 1 - PL/Perl Implementation

Same implementation of Raku: use an hash and iterate over all the matches of a regular expression to get all numbers.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc329.task1_plperl( text )
RETURNS SETOF int
AS $CODE$

   my ( $string ) = @_;
   die "Invalid string" unless( $string =~ / ^ [a-zA-Z0-9]+ $ /x );

   my $found = {};

   for ( $string =~ / [0-9]+ /xg ) {
        $found->{ $_ }++;
   }

   return [ sort keys $found->%* ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 329 - Task 2 - PL/Perl Implementation


Same implementation as in Raku, but with the Schwartzian Transform to `grep` the index of the last match of the `$needle`.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc329.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;
   die "Invalid string" unless( $string =~ / ^ [a-zA-Z]+ $ /x );

   my $found;
   my @chars = split //, $string;
   for my $i ( 0 .. $#chars ) {
       my $current = $chars[ $i ];
       my $needle  = uc $current;
       $needle = lc $current if ( $current =~ /[A-Z]/ );
       my $index = 0;

       my $index = ( sort
	          map { $_->[ 1 ] }
	          grep { $_->[ 0 ] eq $needle }
	          map { [ $_, $index++ ] }  @chars[ $i .. $#chars ] )[ -1 ];
       my $match = join( '', @chars[ $i .. $index ] );
       $found = $match if ( ! $found || length( $found ) < length( $match ) );
   }

   return $found;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 329 - Task 1 - PL/PgSQL Implementation

A single query does the trick!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc329.task1_plpgsql( s text )
RETURNS SETOF text
AS $CODE$
   SELECT distinct( v )
   FROM regexp_matches( s, '\d+', 'g' ) v
   ORDER BY 1
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 329 - Task 2 - PL/PgSQL Implementation

Similar implementation to the Raku one, but longer...

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc329.task2_plpgsql( s text )
RETURNS text
AS $CODE$
DECLARE
	current text;
	i int;
	chars text[];
	result text;
	tmp text;
	needle text;
BEGIN

	SELECT regexp_split_to_array( s, '' )
	INTO chars;

	result := '';

	FOR i in 1 .. length( s ) LOOP
	    current := chars[ i ];
	    needle  := upper( current );

	    IF current ~ '[A-Z]' THEN
	       needle := lower( current );
	    END IF;

	    tmp := '';
	    FOR j IN i .. length( s ) LOOP
	    	IF chars[ j ] = needle THEN
		   tmp := array_to_string( chars[ i : j ], '' );
		   IF length( tmp ) > length( result ) THEN
		      result := tmp;
		   END IF;
	       END IF;
	    END LOOP;

	END LOOP;

	RETURN result;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 329 - Task 1 - PostgreSQL PL/Java Implementation

Use a regular expression to match all the digits, and iterate over the matches to insert into an hash.

<br/>
<br/>
```java
    public static final int[] task1_pljava( String string ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc329.task1_pljava" );

		Map<Integer, Integer> numbers = new HashMap<Integer, Integer>();

		Pattern regexp = Pattern.compile( "\\d+" );
		Matcher engine = regexp.matcher( string );
		while ( engine.find() ) {
		    try {
				numbers.put( Integer.parseInt( engine.group() ), 1 );
		    } catch( Exception e ) { }
		}

		int result[] = new int[ numbers.size() ];
		int j = 0;
		for ( int i : numbers.keySet() )
		    result[ j++ ] = i;

		return result;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 329 - Task 2 - PostgreSQL PL/Java Implementation

A different approach: I use two boundaries `begin` and `end` to keep track of the longest substring for a match.


<br/>
<br/>
```java
    public static final String task2_pljava( String string ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc329.task2_pljava" );


		char chars[] = string.toCharArray();

		int begin = 0;
		int end   = 0;

		for ( int i = 0; i < chars.length; i++ ) {
		    char c = chars[ i ];
		    char n = Character.toUpperCase( c );
		    if ( Character.isUpperCase( c ) )
				n = Character.toLowerCase( c );

		    for ( int j = i + 1; j < chars.length; j++ )
				if ( chars[ j ] == n )
					if ( ( end - begin ) < ( j - i ) ) {
					begin = i;
					end   = j;
			    }
		}

		return string.substring( begin, end + 1 );
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 329 - Task 1 - Python Implementation

Use regular expression and get an implementation similar to PL/Perl.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    string = args[ 0 ]

    numbers = []
    pattern = re.compile( r'\d+' )
    for i in pattern.findall( string ) :
        if not i in numbers :
            numbers.append( i )

    return numbers

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 329 - Task 2 - Python Implementation

Same implementation as in PL/Java.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    string = args[ 0 ]

    begin = 0
    end   = 0

    for i in range( 0, len( string ) - 1 ) :
        current = string[ i ]
        needle = current.upper()
        if current.isupper() :
            needle = current.lower()

        for j in range( i + 1, len( string ) ) :
            print( " => ", string[ j ] )
            if string[ j ] == needle :
                if ( end - begin ) < ( j - i ) :
                    begin = i
                    end   = j

    return string[ begin : end + 1 ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
