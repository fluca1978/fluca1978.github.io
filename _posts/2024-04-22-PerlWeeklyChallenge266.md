---
layout: post
title:  "Perl Weekly Challenge 266: words and matrix indexes"
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

# Perl Weekly Challenge 266: words and matrix indexes

This post presents my solutions to the [Perl Weekly Challenge 266](https://perlweeklychallenge.org/blog/perl-weekly-challenge-266/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 266 - Task 1 - Raku](#task1)
- [PWC 266 - Task 2 - Raku](#task2)
- [PWC 266 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 266 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 266 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 266 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 266 - Task 1 in PL/Java](#task1pljava)
- [PWC 266 - Task 2 in PL/Java](#task2pljava)
- [PWC 266 - Task 1 in Python](#task1python)
- [PWC 266 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 266 - Task 1 - Raku Implementation

The first task was about finding words from two given statements so that every word appears only one time in one of the two sentences and does not appear in the other.

<br/>
<br/>
```raku
sub MAIN( Str $left, Str $right ) {
    my $bag-left  = Bag.new( $left.split( / \W+ / ).map( { .lc } ) );
    my $bag-right = Bag.new( $right.split( / \W+ / ).map( { .lc } ) );

    my @found-words;

    @found-words.push: $_ for $bag-left.keys.grep( -> $left,  { $bag-left{ $left } == 1 && ! $bag-right.keys.grep( { $left eq $_ } ) } );
    @found-words.push: $_ for $bag-right.keys.grep( -> $right,  { $bag-right{ $right } == 1 && ! $bag-left.keys.grep( { $right eq $_ } ) } );
    @found-words.join( ', ' ).say;
}

```
<br/>
<br/>

The above is probably a too much expensive implementation.
The idea is to use two `Bag`s to classify and count the number of words for every statement, then to push into `@found-words` array all the words that are keys for one of the bags assuming it appears with a counting of one and does not matches into the keys of the other bag.

A possible alternative and simplest solution, could have been to concatenate the two sentences, split into words, and extract only those words with a counting of one.


<a name="task2"></a>
## PWC 266 - Task 2 - Raku Implementation

Given a suared matrix, ensure that only the diagonal and antidiagonal have non-zero elements.

<br/>
<br/>
```raku
sub MAIN() {

    my @matrix = [1, 0, 0, 2],
                 [0, 3, 4, 0],
                 [0, 5, 6, 0],
                 [7, 0, 0, 1],
    ;


    my @indexes;
    my $size = @matrix[ 0 ].elems;
    for 0 ..^ $size {
		@indexes.push: [ $_, $_  ];            # main diagonal
		@indexes.push: [ $_, $size - $_ - 1 ]; # antidiagonal
    }

    my $row-index = 0;
    for @matrix -> $row {
		my @zeros = $row.grep( { $_ != 0 }, :k ).map( { [ $row-index, $_ ] } );

		say 'False' and exit if @zeros.elems != $size / 2;
		say 'False' and exit if @indexes.grep( * eq any( @zeros ) ).elems != @zeros.elems;

		$row-index++;
    }

    'True'.say;
}

```
<br/>
<br/>


The above implementation is probably too complex: I create an `@indexes` array with the positions of the main diagonal and anti-diagonal.
then I iterate over the `@matrix` elements and checks where the `@zeros` elements are. If the zeros are at indexes that are not fully matched into the `@indexes` array it means that the matrix has zeros in the diagonal or anti-diagonal, thus it is not good for the aim.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 266 - Task 1 - PL/Perl Implementation

I use two bags, similarly to what happens in Raku. However, this time I return all the words from a bag that do not appear in the other sentence (so not checking the other bag).

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc266.task1_plperl( text, text )
RETURNS SETOF text
AS $CODE$

   my ( $left, $right ) = @_;

   my $bag_left = {};
   my $bag_right = {};

   $bag_left->{ lc $_ }++ for ( split( /\W+/, $left ) );
   $bag_right->{ lc $_ }++ for ( split( /\W+/, $right ) );

   for my $word ( keys $bag_left->%* ) {
       next if $bag_left->{ $word } != 1;
       next if $right =~ /$word/i;

       return_next( $word );
   }

   for my $word ( keys $bag_right->%* ) {
       next if $bag_right->{ $word } != 1;
       next if $left =~ /$word/i;

       return_next( $word );
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 266 - Task 2 - PL/Perl Implementation

A much simpler approach: I iterate over all the elemens and check if I'm on a diagonal or antidiagonal or not.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc266.task2_plperl( int[][] )
RETURNS boolean
AS $CODE$

   my ( $matrix ) = @_;

   my $size = scalar $matrix->@[ 0 ]->@*;

   for my $row ( 0 .. $size - 1 ) {
       for my $column ( 0 .. $size - 1 ) {
       	     # is this the main diagonal?
	     # is the antidiagonal?
	    if ( $row == $column || $column == $size - $row - 1 ) {
	       return 0 if ( $matrix->@[ $row ]->[ $column ] == 0 );
	    }
	    else {
	    	return 0 if ( $matrix->@[ $row ]->[ $column ] != 0 );
	    }
       }

   }

   return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 266 - Task 1 - PL/PgSQL Implementation

A single query to solve the task.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc266.task1_plpgsql( l text, r text )
RETURNS SETOF text
AS $CODE$

   WITH left_words AS (
   	   SELECT w, count( w ) as c
	   FROM regexp_split_to_table( l, '\W+' ) w
   	   GROUP BY 1
	   )
    , right_words AS (
          SELECT w, count( w ) as c
          FROM regexp_split_to_table( R, '\W+' ) w
          GROUP BY 1
    )

    SELECT w
    FROM left_words
    WHERE w NOT IN ( SELECT w FROM right_words )
    AND c = 1
    UNION
    SELECT w
    FROM right_words
    WHERE w NOT IN ( SELECT w FROM left_words )
    AND c = 1
   ;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The two CTEs provide the words of the sentences with their frequence, then I select all the words from one list that do not appear in the other list union the opposite query.


<a name="task2plpgsql"></a>
## PWC 266 - Task 2 - PL/PgSQL Implementation

Same approach as in PL/Perl, but since arrays are monodimensional, the function accepts the size of the row of the matrix and computes dynamically the index depending on an offset.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc266.task2_plpgsql( s int, matrix int[] )
RETURNS boolean
AS $CODE$
DECLARE
	w int := 0;
BEGIN


	FOR r IN 1 .. s LOOP
	    FOR c IN 1 .. s LOOP
	    	w = ( r - 1 ) * s + c;
	    	IF r = c OR c = s - r + 1 THEN
		   IF matrix[ w ] = 0 THEN
		      RETURN false;
		   END IF;
		ELSE
		   IF matrix[ w ] <> 0 THEN
		      RETURN false;
		   END IF;
		END IF;

	    END LOOP;
	END LOOP;

	RETURN true;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 266 - Task 1 - PostgreSQL PL/Java Implementation

Same approach as in PL/Perl: use two hashes as bags. The `deduplicate` method extracts the list of words that appear only in one bag and not in the other.

<br/>
<br/>
```java
public class Task1 {

	private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc266",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String[] task1_pljava( String left, String right ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc266.task1_pljava" );

		Map<String, Integer> bag_left  = new HashMap<String, Integer>();
		Map<String, Integer> bag_right = new HashMap<String, Integer>();

		classify( bag_left, left );
		classify( bag_right, right );


		List<String> words = deduplicate( bag_left, bag_right );
		String returning[] = new String[ words.size() ];
		int i = 0;
		for ( String word : words )
		    returning[ i++ ] = word;

		return returning;
    }


    private static final List<String> deduplicate( Map<String, Integer> bag_left,
						   Map<String, Integer> bag_right ) {

		List<String> words = new LinkedList<String>();

		for ( String word : bag_left.keySet() ) {
		    if ( bag_left.get( word ) == 1 && ! bag_right.containsKey( word ) )
			words.add( word );
		}

		for ( String word : bag_right.keySet() ) {
		    if ( bag_right.get( word ) == 1 && ! bag_left.containsKey( word ) )
				words.add( word );
		}

	    return words;
    }

    private static final void classify( Map<String, Integer> bag, String sentence ) {
		for ( String word : sentence.split( "\\W+" ) ) {
		    word = word.toLowerCase();
		    int count = 1;
		    if ( bag.containsKey( word ) )
				count += bag.get( word );

		    bag.put( word, count );
		}
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 266 - Task 2 - PostgreSQL PL/Java Implementation

Same approach as in PL/PgSQL: iterate over the matrix, that is a monodimensional array, and check if I'm on the main diagonal or the antidiagonal.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc266",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final boolean task2_pljava( int dim, int[] matrix ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc266.task2_pljava" );

		for ( int row = 0; row < dim; row++ ) {
		    for ( int col = 0; col < dim; col++ ) {

			if ( row == col || col == dim - row - 1 ) {
			    if ( matrix[ row * dim + col ] == 0 )
					return false;
			}
			else if ( matrix[ row * dim + col ] != 0 )
			    return false;

		    }
		}

		return true;

    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 266 - Task 1 - Python Implementation

Same approach as in PL/Perl, with a few more iterations.

<br/>
<br/>
```python
 import sys

# task implementation
# the return value will be printed
def task_1( args ):
    left  = args[ 0 ]
    right = args[ 1 ]

    bag_left  = {}
    bag_right = {}

    # classify
    for w in left.split():
        c = 1
        w = w.lower()
        if w in bag_left:
            c += bag_left[ w ]

        bag_left[ w ] = c

    for w in right.split():
        c = 1
        w = w.lower()
        if w in bag_right:
            c += bag_right[ w ]

        bag_right[ w ] = c

    words = []
    for w in bag_left:
        if bag_left[ w ] == 1 and not w in bag_right:
            words.append( w )

    for w in bag_right:
        if bag_right[ w ] == 1 and not w in bag_left:
            words.append( w )

    return words


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 266 - Task 2 - Python Implementation

Same approach as in PL/Java.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( matrix ):
    size = len( matrix[ 0 ] )
    for row in range( 0, size ):
        for col in range( 0, size ):
            if row == col or col == size - row - 1:
                if matrix[ row ][ col ] == 0:
                    return 'False'
            elif matrix[ row ][ col ] != 0:
                return 'False'
    return 'True'



# invoke the main without the command itself
if __name__ == '__main__':
    matrix = [
        [1, 0, 0, 2],
        [0, 3, 4, 0],
        [0, 5, 6, 0],
        [7, 0, 0, 1],
    ]
    print( task_2( matrix ) )

```
<br/>
<br/>
