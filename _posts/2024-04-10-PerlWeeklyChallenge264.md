---
layout: post
title:  "Perl Weekly Challenge 264: array indexes mess"
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

# Perl Weekly Challenge 264: array indexes mess

This post presents my solutions to the [Perl Weekly Challenge 264](https://perlweeklychallenge.org/blog/perl-weekly-challenge-264/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 264 - Task 1 - Raku](#task1)
- [PWC 264 - Task 2 - Raku](#task2)
- [PWC 264 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 264 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 264 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 264 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 264 - Task 1 in PL/Java](#task1pljava)
- [PWC 264 - Task 2 in PL/Java](#task2pljava)
- [PWC 264 - Task 1 in Python](#task1python)
- [PWC 264 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 264 - Task 1 - Raku Implementation

The first task was, given a string made only of letters, to find out the *biggest* letter, that is a letter uppercase that appears in the string also as lowecase and that is the last one in the alphabet sorting.

<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / ^ <[a..zA..Z]>+ $ / } ) {
    my @letters;

    for $string.comb.grep( * ~~ / <[A..Z]> / ) -> $letter {
		next if ! $string ~~ / $letter.lc /;
		@letters.push: $letter;
    }

    @letters.sort[ * - 1 ].say if ( @letters );
}

```
<br/>
<br/>

The idea is simple: I iterate over the string letters, and then consider only the uppercase ones that appear as lowercase too.
If an uppercase letter appears also as lowercase, I add it to the `@letters` array, that then I sort to print out the very last entry.


<a name="task2"></a>
## PWC 264 - Task 2 - Raku Implementation

The second task was to produce an array, given two input ones, where the former is of values and the second is of indexes. The idea is to produce the third array pushing repeated elements from the indexes.

<br/>
<br/>
```raku
sub MAIN( :@source, :@indexes
		      where { @source.elems == @indexes.elems
						&& @indexes.min >= 0
						&& @indexes.max <= @source.elems } ) {

    my @target;

    for 0 ..^ @indexes.elems  {
		my $target-index = @indexes[ $_ ];
		if ! @target[ $target-index ] {
		    @target[ $target-index ] = @source[ $_ ];
		} else {
		    @target = |@target[ 0 .. $target-index - 1 ], @source[ $_ ], |@target[ $target-index  .. * ];
		}
    }

    @target.join( ' ' ).say;
}

```
<br/>
<br/>

The idea is that, given an index, I compute the `$target-index` that is the value of the `@target` array given the array of indexes. If the element does not exist in the `@target` array, then I simply need to append the value, otherwise I need to slice the array, inserting the value in the middle.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 264 - Task 1 - PL/Perl Implementation

Essentially, the same approach used in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc264.task1_plperl( text )
RETURNS char
AS $CODE$

   my ( $string ) = @_;
   die "Must contain only letters" if ( ! $string =~ /^[a-zA-Z]+$/ );

   my @letters;
   for my $letter ( grep( { $_ =~ /[A-Z]/ } split( //, $string ) ) ) {
       my $lc_letter = lc $letter;
       next if $string !~ /$lc_letter/;
       push @letters, $letter;
   }

   return '' if ! @letters;
   return ( sort( @letters ) )[ -1 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 264 - Task 2 - PL/Perl Implementation

Same slicing approach used in Raku. Please note how complex is to check for the input values.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc264.task2_plperl( int[], int[] )
RETURNS int[]
AS $CODE$

   my ( $source, $indexes ) = @_;

   die "Array must have the same size!" if ( scalar( $source->@* ) != scalar( $indexes->@* ) );
   for ( $indexes->@* ) {
       next if $_ >= 0 && $_ <= scalar( $source->@* );
       die "Array indexes must be contained in the source array"
   }


   my @target;

   for my $index ( 0 .. $indexes->@* - 1 ) {
       my $target_index = $indexes->@[ $index ];

       if ( ! $target[ $target_index ] ) {
       	  push @target, $source->@[ $index ];
       }
       else {
       	    @target = ( @target[ 0 .. $target_index - 1  ], $source->@[ $index ], @target[ $target_index .. $#target ] );
       }
   }

   return [ @target ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 264 - Task 1 - PL/PgSQL Implementation

I use a temporary table to store each uppercase letter with a flag that indicates if the letter has appearead at least once as lowercase.



<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc264.task1_plpgsql( s text )
RETURNS char
AS $CODE$
DECLARE
	letter char;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS letters( l char, ok boolean default false );
	TRUNCATE letters;

	INSERT INTO letters
	SELECT v
	FROM regexp_split_to_table( s, '' ) v
	WHERE v ~ '[A-Z]'
	GROUP BY v;

	FOR letter IN SELECT v::char FROM regexp_split_to_table( s, '[a-z]' ) v LOOP
	    UPDATE letters
	    SET ok = true
	    WHERE ok = false
	    AND l = upper( letter );
	END LOOP;

	SELECT l
	INTO letter
	FROM letters
	WHERE ok
	ORDER BY 1 DESC
	LIMIT 1;

	RETURN letter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

In the beginning, I add every uppercase letter to the temporary table. Then I iterate over all the lowercase letters and update the `ok` flag in the table for the corresponding uppercase letter. Last, I select the only last one letter in the table with the `ok` flag.


<a name="task2plpgsql"></a>
## PWC 264 - Task 2 - PL/PgSQL Implementation

I use a temporary table to store every value with its *needed* recomputed index.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc264.task2_plpgsql( s_a int[], i_a int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	target_index int;
	current_index int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS target( i int, v int );
	TRUNCATE target;

	FOR current_index IN SELECT v::int FROM unnest( i_a ) v LOOP
	    target_index := i_a[ current_index ];

	    PERFORM *
	    FROM target
	    WHERE i = target_index;

	    IF FOUND THEN
				UPDATE target
				SET i = i + 1
				WHERE i >= target_index;

	    END IF;

	    INSERT INTO target( i, v )
	    VALUES ( target_index, s_a[ current_index ] );

	END LOOP;

	RETURN QUERY
	SELECT v
	FROM target
	ORDER BY i;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

If a computed `target_index` is found in the table, all the tuples with an index value greater or equal ar shifted by one unit. Then the current value is inserted, and at last the query returns the values sort by their indexes.

# Java Implementations

<a name="task1pljava"></a>
## PWC 264 - Task 1 - PostgreSQL PL/Java Implementation

The same approach used in PL/Perl: I keep a list of uppercase letters that is populated only if the letter appears also as lowercase.
The list is then sorted and the last value is extracted.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc264",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task1_pljava( String string ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc264.task1_pljava" );

		List<String> letters = new LinkedList<String>();
		for ( String l : string.split( "" ) ) {
		    if ( ! l.equals( l.toUpperCase() ) )
			 continue;

		    Pattern regexp = Pattern.compile( l.toLowerCase() );
		    Matcher engine = regexp.matcher( string );
		    if ( ! engine.matches() )
			letters.add( l );
		}

		if ( letters.isEmpty() )
		    return "-";

		Collections.sort( letters );
		return letters.get( letters.size() - 1 );
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 264 - Task 2 - PostgreSQL PL/Java Implementation

The approach is similar to the slicing already performed in Raku and PL/Perl, but since I don't know about a way to slice arrays and lists in Java, I sued two loops to produce the same result.


<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc264",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task2_pljava( int[] source, int[] indexes ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc264.task2_pljava" );

		List<Integer> target = new LinkedList<Integer>();

		for ( int i = 0; i < indexes.length; i++ ) {
		    int target_index = indexes[ i ];
		    if ( target.isEmpty() ||  target.size() <= target_index )
				target.add( source[ i ] );
		    else {
				List<Integer> replace = new LinkedList<Integer>();
				for ( int ii = 0; ii < target_index; ii++ )
					replace.add( target.get( ii ) );
				replace.add( source[ i ] );
				for ( int ii = target_index; ii < target.size(); ii++ )
					replace.add( target.get( ii ) );

		       target = replace;
		    }
		}

		int[] result = new int[ target.size() ];
		int k = 0;
		for ( int v : target )
		    result[ k++ ] = v;

		return result;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 264 - Task 1 - Python Implementation

I iterate over all the letters in the given string, and skip it if it is not an uppercase one.
In the case it is, I compile a regular expression to see if the letter appears in the string as lowercase, and if there is a match I add the letter to the `letters` array, that is then sorted and the last element is returned.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    string = args[ 0 ]
    letters = []

    for letter in string:
        if not letter.isupper():
            continue

        engine = re.compile( letter.lower() )
        if not engine.search( string ):
            continue

        letters.append( letter )

    if len( letters ) <= 0:
        return ''

    letters.sort()
    return letters[ len( letters ) - 1 ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 264 - Task 2 - Python Implementation

A quite verbose implementation, needed also because I use an input string pipe to separate the two arrays on the command line.

The slicing is performed by using an intermediate array and two separated slices, similarly to what I've done in Java but without loops.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    source  = []
    indexes = []
    target  = []
    swap    = False
    for v in args:
        if v == '|':
            swap = True
            continue
        else:
            if swap:
                indexes.append( int( v ) )
            else:
                source.append( int( v ) )


    for i in range( 0, len( indexes ) ):
        target_index = indexes[ i ]
        if len( target ) <= target_index:
            target.insert( target_index, source[ i ] )
        else:
            swapper = []
            swapper = target[ 0 : target_index ]
            swapper.append( source[ i ] )
            swapper += target[ target_index : ]
            target = swapper


    return target

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
