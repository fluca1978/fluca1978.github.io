---
layout: post
title:  "Perl Weekly Challenge 260: Occurrencies and Permutations"
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

# Perl Weekly Challenge 260: Occurrencies and Permutations

This post presents my solutions to the [Perl Weekly Challenge 260](https://perlweeklychallenge.org/blog/perl-weekly-challenge-260/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 260 - Task 1 - Raku](#task1)
- [PWC 260 - Task 2 - Raku](#task2)
- [PWC 260 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 260 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 260 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 260 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 260 - Task 1 in PL/Java](#task1pljava)
- [PWC 260 - Task 2 in PL/Java](#task2pljava)
- [PWC 260 - Task 1 in Python](#task1python)
- [PWC 260 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 260 - Task 1 - Raku Implementation

The first task was to find out if, in a given list of numbers, every number has an occurency count unique.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    my $bag = Bag.new( @nums );

    for $bag.values -> $current {
		'0'.say and exit if ( $bag.values.grep( * ~~ $current ).elems > 1 );
    }

    '1'.say;

}

```
<br/>
<br/>

The idea is to store into a `Bag` the number with their occurrencies, and then to see how many times the values (i.e., the occurrencies) are repeated within the `$bag` values.


<a name="task2"></a>
## PWC 260 - Task 2 - Raku Implementation

This is one case where I find the second task simpler than the first one.
Given a word, compute its *dictionary classification (rank)* that is computed as the index (numbered from one) in the list of all the sorted permutations of the letters of the word.

<br/>
<br/>
```raku
sub MAIN( $word ) {
    say $word.comb.permutations.map( *.join ).sort.grep( * ~~ $word, :k ).first + 1;
}

```
<br/>
<br/>

I first split the `$word` into letters by means of `comb`, then compute all the `permutations` and remape every array of letters into a word, `sort` the list of words and `grep` the initial word into this list, extracting only the index. The `first + 1` is the way I obtain only the value of the index and add one.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 260 - Task 1 - PL/Perl Implementation

I use a `$bag` hash to store every number with the count of its occurrencies, but storing only once for every repretition of the number.

Then I iterate over the values of the hash, to see if there is more than one value repeated.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc260.task1_plperl( int[] )
RETURNS boolean
AS $CODE$

   my ( $nums ) = @_;
   my $bag = {};

   for my $current ( $nums->@* ) {
       next if $bag->{ current }; # no need to reinitialize
       $bag->{ $current } = scalar grep { $current == $_ } $nums->@*;
   }

   for my $current ( values $bag->%* ) {
       return 0 if ( scalar( grep { $current == $_ } values( $bag->%* ) ) > 1 );
   }

   return 1;

$CODE$
LANGUAGE plperl;
```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 260 - Task 2 - PL/Perl Implementation

I use `List::Permutor` to obtain all the permutations of the array of letters, and then pushing the join result into a words list.
Last, I search into the list the position of the given word.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc260.task2_plperl( text )
RETURNS int
AS $CODE$
   use List::Permutor;

   my ( $word ) = @_;
   my @words;

   my $engine = List::Permutor->new( split //, $word );
   while ( my @letters = $engine->next ) {
   	 push @words, join( '', @letters );
   }

   @words = sort @words;


   for my $index ( 0 .. @words - 1 ) {
       return $index + 1 if ( $words[ $index ] eq $word );
   }

   return -1;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 260 - Task 1 - PL/PgSQL Implementation

A single query can do the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc260.task1_plpgsql( nums int[] )
RETURNS boolean
AS $CODE$

   WITH numbers AS (
   	SELECT n
	FROM unnest( nums ) n
	)
   , counting AS (
	SELECT n, count(*) AS c
	FROM numbers
	GROUP BY n
	)
   , grouping AS (
     	SELECT c, count(*) AS g
	FROM counting
	GROUP BY c
	)
   SELECT NOT EXISTS( SELECT g
   	      	      FROM grouping
   		      WHERE g > 1
		      );

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The common table expression is built in incremental way:
- `numbers` provides a table with the given list of numbers;
- `counting` counts how many times the same number appears in the list;
- `grouping` counts how many times the counting appears.

Therefore, it does only suffice to select if a tuple has a *grouping* greater than one to know if there are occurrencies repetitions.


<a name="task2plpgsql"></a>
## PWC 260 - Task 2 - PL/PgSQL Implementation

I cheated here, using PL/Perl, since permutations in SQL are too time wasting in my opinion.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc260.task2_plpgsql( w text )
RETURNS int
AS $CODE$
   SELECT pwc260.task2_plperl( w );
$CODE$
LANGUAGE ùsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 260 - Task 1 - PostgreSQL PL/Java Implementation

There is more code to translate types than to do the real work!

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc260",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final boolean task1_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc260.task1_pljava" );

		List<Integer> numList = new LinkedList<Integer>();
		for ( int n : nums )
		    numList.add( n );

		Map<Integer, List<Integer>> bag = new HashMap<Integer, List<Integer>>();

		for ( int current : numList ) {
		    int occurrency = Collections.frequency( numList, current );
		    bag.putIfAbsent( occurrency, new LinkedList<Integer>() );
		    if ( ! bag.get( occurrency ).contains( current ) )
				bag.get( occurrency ).add( current );
		}

		for ( int k : bag.keySet() )
		    if ( bag.get( k ).size() > 1 )
			return false;


		return true;

    }
}

```
<br/>
<br/>


The idea is the same as in PL/Perl: `bag` is used to classify the numbers (uniquely) with their occurrencies, that is computed by means of `Collections.frequency()`. For every `occurrency` I place into the `bag` the list of numbers that have such frequency. Then, I search if there's a list in the `bag` that has more than one element, and if found, I return a `false` value.

<a name="task2pljava"></a>
## PWC 260 - Task 2 - PostgreSQL PL/Java Implementation

Doing permutations in Java is such a mess!


<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc260",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( String word ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc260.task2_pljava" );

		List<String> chars = new LinkedList<String>();
		for ( String s : word.split( "" ) ) {
		    logger.log( Level.INFO, "CHAR " + s );
		    chars.add( s );
		}

		List<String> words = new LinkedList<String>();

		BigInteger limit = BigInteger.ONE;
		for ( int i = 1; i <= chars.size(); i++ ) {
		    limit = limit.multiply( new BigInteger( "" + i ) );
		}

		while ( BigInteger.ZERO.compareTo( limit ) < 0 ) {


		    String newWord = "";
		    do {
				Collections.shuffle( chars );
				newWord = String.join( "", chars );
		    } while ( words.contains( newWord ) );

		    words.add( newWord );
		    limit = limit.subtract( BigInteger.ONE );
		}

		Collections.sort( words );

		for ( int i = 0; i < words.size(); i++ )
		    if ( words.get( i ).equals( word ) )
			return i + 1;


		return -1;
    }
}

```
<br/>
<br/>

I first split the `word` into a list of characters, named `chars`. Then I compute the `limit` in terms of possible permutations, and I use a `BigInteger` because a factorial number could quickly become very large.
The next step is to `shuffle` the list of characters, so that they are randomly arranged. **This is not the same as computing all the possible permutations** since `shuffle()` could return the same value over and over, so I need to add the shuffled word to the list of words only if not already placed, and only then decrease the `limit` value.

Last is the same as in other implementations: sorting the list and searching for an entry that matches the inital word.


# Python Implementations

<a name="task1python"></a>
## PWC 260 - Task 1 - Python Implementation

Use the same bag approach used in Raku.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    nums = list( map( int, args ) )
    bag = {}

    for n in nums:
        occurrencies = len( list( filter( lambda x: x == n, nums ) ) )
        if not n in bag:
            bag[ n ] = occurrencies


    for o in bag.values():
        if len( list( filter( lambda x: x == o, bag.values() ) ) ) > 1:
            return 0

    return 1


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 260 - Task 2 - Python Implementation

Luckily, Python has a way to create permutations!

<br/>
<br/>
```python
import sys
import array
from itertools import permutations

# task implementation
# the return value will be printed
def task_2( args ):
    word = args[ 0 ]
    words = []
    engine = permutations( list( word ) )
    for p in list( engine ):
        words.append( ''.join( p ) )

    words = sorted( words )
    for i in range( 0, len( words ) ):
        if words[ i ] == word:
            return i + 1


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>

The idea is to create the permutations of characters of the original word, re-joining them and adding to a `words` list. Then, as for other implementations, sort and search for the position of the initial word.
