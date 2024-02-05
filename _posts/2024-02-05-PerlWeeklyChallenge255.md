---
layout: post
title:  "Perl Weekly Challenge 255: Banned Words and Exceeding Letters"
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

# Perl Weekly Challenge 255: Banned Words and Exceeding Letters

This post presents my solutions to the [Perl Weekly Challenge 255](https://perlweeklychallenge.org/blog/perl-weekly-challenge-255/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 255 - Task 1 - Raku](#task1)
- [PWC 255 - Task 2 - Raku](#task2)
- [PWC 255 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 255 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 255 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 255 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 255 - Task 1 in PL/Java](#task1pljava)
- [PWC 255 - Task 2 in PL/Java](#task2pljava)
- [PWC 255 - Task 1 in Python](#task1python)
- [PWC 255 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 255 - Task 1 - Raku Implementation

The first task was about finding out a letter that mismatches between an original word and a shuffled one.

<br/>
<br/>
```raku
sub MAIN( Str :$a, Str :$b where { $b.chars == $a.chars + 1 } ) {
    my $classification = BagHash.new: $b.comb;
    for $a.comb {
		$classification{ $_ }--;
		$classification{ $_ }:delete if ( $classification{ $_ } <= 0 );
    }

    $classification.keys.head.say;
}

```
<br/>
<br/>

At glance I thought it would suffice to use a good concatenation of `comb` and `grep`, but the task was requiring for detection of even duplicated values. Therefore I decided to use a `Bag`, or better a `BagHash` (hence, modificable), to store a counting of how many occurrencies of a char appear in the shuffled string. Then, for every character matching between the two string I decrease the counter, and if it hits zero, I totally remove the entry from the hash.
Therefore, only one char should have remained as key in the bag, and I can then extract it and print out.



<a name="task2"></a>
## PWC 255 - Task 2 - Raku Implementation

The second task was about finding out the most occurring word in a bunch of text, ecluding a banned word.

<br/>
<br/>
```raku
sub MAIN( Str :$p is copy, Str :$w ) {

    $p ~~ s:g/ \W* $w \W+/ /;
    my $classification = Bag.new: $p.split( / \W+ /, :skip-empty );
    $classification.keys.grep( { $classification{ $_ } == $classification.values.max } ).first.say;

}

```
<br/>
<br/>

The idea is, as first step, to remove every occurrency of the banned word from the paragraph of text.
Then I use a `Bag` to store the result of splitting the text on the word boundaries, so to get a counting of every word with its occurrencies.
Last, I extract the max occurrency and search for the first word with such counting to print.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 255 - Task 1 - PL/Perl Implementation

The approach is pretty much the same as in Raku: I use a `$classification` hash to count the occurrencies of every letter in the shuffled string, decreasing the count for every letter of the original string. Then, I iterate over the keys of the hash to find out the first one that has a non-zero value.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc255.task1_plperl( text, text )
RETURNS text
AS $CODE$

   my ( $origin, $shuffled ) = @_;
   die "Shuffled string [$shuffled] should be one char lengther than original [$origin]"
       if ( length( $shuffled ) != length( $origin ) + 1 );


  my $classification = {};
  $classification->{ $_ }++ for ( split //, $shuffled );
  $classification->{ $_ }-- for ( split //, $origin );

  for ( keys $classification->%* ) {
      return $_ if ( $classification->{ $_ } > 0 );
  }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 255 - Task 2 - PL/Perl Implementation

Same approach as in Raku: use a `$classification` hash keyed by every word and counting the occurrencies. Before doing the count, I remove the banned word with a regular expression. The `$max` value (occurrency) is computed sorting the values and extracting the topmost, then I iterate over the `$classification` hash and extract only the topmost key (word).

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc255.task2_plperl( text, text )
RETURNS text
AS $CODE$

   my ( $paragraph, $banned ) = @_;

   $paragraph =~ s/\W*$banned\W*/ /g;

   my $classification = {};
   $classification->{ $_ }++  for ( split /\W/, $paragraph );
   my $max = ( reverse sort values $classification->%* )[ 0 ];
   for ( keys $classification->%* ) {
       return $_ if ( $classification->{ $_ } == $max );
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 255 - Task 1 - PL/PgSQL Implementation

I use a single query to find out any mismatching character here.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc255.task1_plpgsql( origin text, shuffled text )
RETURNS char
AS $CODE$
   SELECT s
   FROM regexp_split_to_table( shuffled, '' ) s
   WHERE s NOT IN ( SELECT v
                    FROM regexp_split_to_table( origin, '' ) v );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The problem of this approach is that it cannot detect a mismatching character unless it is different from any character of the original string.


<a name="task2plpgsql"></a>
## PWC 255 - Task 2 - PL/PgSQL Implementation

I use a temporary table to solve this task.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc255.task2_plpgsql( p text, b text )
RETURNS SETOF text
AS $CODE$
DECLARE

BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS classification( w text );
	TRUNCATE TABLE classification;

	INSERT INTO classification
	SELECT v
	FROM regexp_split_to_table( p, '\W+' ) v;

	RETURN QUERY
	       SELECT w FROM (
	       	 SELECT w, count(*) AS c
	       	 FROM classification
	       	 WHERE w <> b
	       	 GROUP BY w
	       	 ORDER BY c DESC
	       	 LIMIT 1
	       );

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The `classification` table contains a row for every word fount in the text. Then I extract the tompost occurring word that is not banned and return it with a nested query.


# Java Implementations

<a name="task1pljava"></a>
## PWC 255 - Task 1 - PostgreSQL PL/Java Implementation

Similarly to the Perl solutions, I use a `classification` hash to count every character of the shuffled string. Later, I decrease every occurrency for every match with a character of the original string, and extract the topmost remaining character.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
    public static final String task1_pljava( String origin, String shuffled ) throws SQLException {
		logger.log( Level.INFO, "Entering task1_pljava" );

		if ( origin.length() + 1 != shuffled.length() ) {
		    throw new SQLException( "Shuffled string should be one character longer than the original one" );
		}

		Map<String, Integer> classification = new HashMap<String, Integer>();
		for ( String needle : shuffled.split( "" ) ) {
		    int value = classification.keySet().contains( needle )
				? classification.get( needle )
				: 0;

		    classification.put( needle, ++value );
		}


		for ( String comparison : origin.split( "" ) ) {
		    int value = classification.get( comparison );
		    value--;

		    if ( value > 0 )
				classification.put( comparison, value );
		    else
				classification.remove( comparison );
		}


		return classification.keySet().iterator().next();

    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 255 - Task 2 - PostgreSQL PL/Java Implementation

Same approach as other solutions: I use a `classification` hash to store the occurrencies of not banned words.
Then, using the stream API, I extract the topmost occurring word. **This is the first time I use the Java Stream API!**

<br/>
<br/>
```java
public class Task2 {
    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
    public static final String task2_pljava( String paragraph, String banned ) throws SQLException {
		logger.log( Level.INFO, "Entering task2_pljava" );

		Map<String, Integer> classification = new HashMap<String, Integer>();
		for ( String word : paragraph.split( "\\W+" ) ) {
		    if ( word.equals( banned ) )
			continue;

		    int value = classification.containsKey( word ) ? classification.get( word ) : 0;
		    classification.put( word, ++value );
		}

		String topValue = classification.entrySet()
		    .stream()
		    .sorted( Collections.reverseOrder( Map.Entry.comparingByValue() ) )
		    .findFirst().get().getKey();

		return topValue;
    }
}

```
<br/>
<br/>

The `topValue` is extracted by getting the `stream()` out of the contents (`entrySet()`) of the `classification` map. Then I use `sorted()` to sort the stream, and `Collections` already provide a `reverseorder` method and a `comparingByvalue()` way to sort. I then apply `findFirst()` to get the first element, and then `get()` to extract it (i.e., a `Map<String, Integer>`) from which I can get the word by means of `getKey()`.

# Python Implementations

<a name="task1python"></a>
## PWC 255 - Task 1 - Python Implementation

Same idea as other implementatins: using a `classification` dictionary as a counter of letter occurrencies, decreasing every letter counting for a matching char.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    origin = args[ 0 ]
    shuffled = args[ 1 ]
    if len( shuffled ) != len( origin ) + 1:
        return "Shuffled string must be one character longer than original string"

    classification = {}

    for needle in shuffled:
        if not needle in classification:
            classification[ needle ] = 0

        classification[ needle ] += 1

    for needle in origin:
        classification[ needle ] -= 1
        if classification[ needle ] <= 0:
            del classification[ needle ]

    return list( classification.keys() )[ 0 ]


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>

As a result, the first key of the dictionary is the one to return.


<a name="task2python"></a>
## PWC 255 - Task 2 - Python Implementation

I use a `classification` dictionary to count every word occurrency, skipping the banned word.
Then I sort the dictionary by means of `sorted`, getting the last `[ -1 ]` element (i.e, the one with the topmost value) and from such item the first entry, i.e., the key (word).

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    paragraph = args[ 0 ]
    banned    = args[ 1 ]
    classification = {}
    for w in paragraph.split():
        if w != banned:
            if not w in classification:
                classification[ w ] = 0
            classification[ w ] += 1

    return sorted( classification.items() )[ -1 ][ 0 ]

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
