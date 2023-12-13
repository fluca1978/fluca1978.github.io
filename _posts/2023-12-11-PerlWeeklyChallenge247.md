---
layout: post
title:  "Perl Weekly Challenge 247: arrays everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 247: arrays everywhere!

This post presents my solutions to the [Perl Weekly Challenge 247](https://perlweeklychallenge.org/blog/perl-weekly-challenge-247/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 247 - Task 1 - Raku](#task1)
- [PWC 247 - Task 2 - Raku](#task2)
- [PWC 247 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 247 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 247 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 247 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 247 - Task 1 in Python](#task1python)
- [PWC 247 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 247 - Task 1 - Raku Implementation

The first task, *that I understood only partially*, was to provide a way to randomly assign a single present from one person to another. It is not clear to me the exact way, so I simply implemented a random way so that a person can give a gift and only one to another person different from herself.

<br/>
<br/>
```raku
sub MAIN( *@names where { @names.grep( * ~~ / ^ M (r||rs||iss) \. .* $ / ).elems == @names.elems } ) {

    my @santa = @names;
    my @receiving = @names;
    my @pairs;


    while ( @pairs.elems < @names.elems ) {
	for @santa.pick -> $santa {
	    next if @pairs.grep( { $_[ 0 ] ~~ $santa } );
	    for @receiving.pick -> $giving {
				next if $santa ~~ $giving;
				next if @pairs.grep( { $_[ 1 ] ~~ $giving } );
				@pairs.push: [ $santa, $giving ];
	    }
	}
    }

    "%s -> %s\n".printf( $_[ 0 ], $_[ 1 ] ) for @pairs;
}

```
<br/>
<br/>

The idea is to keep a `@pairs` array that stores the associations between a `$santa` and a `$giving` receiver. I skip the association if `$santa` and `$giving` are the same person, and continue unless the `pairs` is complete.


<a name="task2"></a>
## PWC 247 - Task 2 - Raku Implementation

The second task was about finding the most frequent couple of subsequent letters in a given string.

<br/>
<br/>
```rakusub MAIN( Str $string where { $string ~~ / ^ <[a .. z ]>+ $ / } ) {

    my @letters = $string.comb;
    my %score;
    for 0 ..^ @letters.elems - 1 {
		my $needle = @letters[ $_ ] ~ @letters[ $_ + 1 ];
		$string ~~ m:g/ $needle /;
		%score{ $/.elems }.push: $needle;
    }

    ( %score{ ( %score.keys.sort )[ * - 1 ] }.sort )[ 0 ].say;
}


```
<br/>
<br/>


I build the searching for string `$needle` with a couple of subsequent letters, then I match against the original string and count the `$/.elems` matches, pushing them into an hash. Then I took the hash key with the highest value, extract the array of needles, sort them and keep the first one.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 247 - Task 1 - PL/Perl Implementation

The basic idea is similar to the Raku implementation, but this time I don't need to store the pairs and I return them immediatly.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc247.task1_plperl( text[] )
RETURNS TABLE( santa text, receiver text )
AS $CODE$
   my ( $names ) = @_;

   my @santas = $names->@*;
   my @receivers = $names->@*;

   while ( @santas > 0 && @receivers > 0 ) {
      my ( $s, $r ) = ( int( rand( scalar( @santas ) ) ), int( rand( scalar( @receivers ) ) ) );
      my $santa = @santas[ $s ];
      my $receiver = @receivers[ $r ];
      next if ! $santa || ! $receiver || $santa eq $receiver;

     return_next( { santa => $santa,
       		        receiver => $receiver } );
     delete @santas[ $s ];
     delete @receiver[ $r ];
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Since I cannot repeat a pair, I delete the people from their array. *This could be wrong, but it is the part not clear to me about the requirements: if a person receive her gift, the person is removed from the list, so she will never play as a Santa.*


<a name="task2plperl"></a>
## PWC 247 - Task 2 - PL/Perl Implementation

Same idea of Raku, but a slightly different implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc247.task2_plperl( text )
RETURNS TABLE( needle text, repetitions int )
AS $CODE$
   my ( $string ) = @_;

   my @letters = split //, $string;
   my $score = {};

   for ( 0 .. @letters - 2 ) {
       my $needle = $letters[ $_ ] . $letters[ $_ + 1 ];
       my @matches = ( $string =~ / $needle /xg );
       my $count = scalar @matches;
       push $score->{ $count }->@*, $needle;
   }

   my $best = ( reverse sort keys $score->%* )[ 0 ];
   return_next( { needle => ( sort $score->{ $best }->@* )[ 0 ],
   		          repetitions => $best } );

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 247 - Task 1 - PL/PgSQL Implementation

A much more verbose implementation: the idea is the same, but I use two temporary tables to store the list of names and removing them once they have been giving and receiving.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc247.task1_plpgsql( n text[] )
RETURNS TABLE( current_santa text, current_receiver text )
AS $CODE$

DECLARE
	remaining int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS santas( santa text );
	CREATE TEMPORARY TABLE IF NOT EXISTS receivers( receiver text );
	TRUNCATE santas;
	TRUNCATE receivers;

	INSERT INTO santas
	SELECT unnest( n );

	INSERT INTO receivers
	SELECT unnest( n );

	SELECT count(*)
	INTO remaining
	FROM santas;

	WHILE remaining > 0 LOOP

		SELECT santa
		INTO current_santa
		FROM santas
		ORDER BY random()
		LIMIT 1;


		SELECT	receiver
		INTO current_receiver
		FROM receivers
		ORDER BY random()
		LIMIT 1;


		IF current_receiver = current_santa THEN
		   CONTINUE;
		END IF;

		RETURN NEXT;

		DELETE FROM santas    WHERE santa = current_santa;
		DELETE FROM receivers WHERE receiver = current_receiver;

		SELECT count(*)
		INTO remaining
		FROM santas;

	END LOOP;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


Every time a valid pair is created, a `RETURN NEXT` ensures the tuple is outputted, and then the people are deleted from the respective table. Again, this could be wrong, but I confess I've not understood the problem requirements.

<a name="task2plpgsql"></a>
## PWC 247 - Task 2 - PL/PgSQL Implementation

I use the regexp capabilities to count the matches of ad-hoc built strings.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc247.task2_plpgsql( string text )
RETURNS TABLE( needle text, repetitions int )
AS $CODE$

DECLARE
	needle text;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS repetitions( needle text, repetition int );
	TRUNCATE repetitions;

	FOR i IN 1 .. length( string ) LOOP
	    needle := substr( string, i, 2 );

	    INSERT INTO repetitions
	    SELECT needle, ( SELECT count(*) FROM  regexp_matches( string, needle, 'g' )f );

	END LOOP;

	RETURN QUERY
	SELECT *
	FROM repetitions
	ORDER BY repetition DESC
	LIMIT 1;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The `needle` string is composed by two chars placed at the offset `i`, which in turn ranges from the very first char index to the last one. The temporary table `repetitions` stores the needle and the count of the global matches, so that it does suffice to return the appropriate query out of the `repetitions` table.


# Python Implementations

<a name="task1python"></a>
## PWC 247 - Task 1 - Python Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```python
import sys
from random import randrange, choice

# task implementation
def main( argv ):
    santas    =  argv.copy()
    receivers =  argv.copy()

    while len( santas ) >= 0 :
        current_santa = santas[ randrange( 0, len( santas )  ) ]
        current_receiver = receivers[ randrange( 0, len( receivers )  ) ]

        if ( current_santa == current_receiver ):
            continue

        print( current_santa, " -> ", current_receiver )
        santas.remove( current_santa )
        receivers.remove( current_receiver )

        if len( santas ) == 0:
            break


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 247 - Task 2 - Python Implementation

The idea is the same as in the other implementations: use a dictionary as an hash keyed by the counting of the occurrencies, and with values that are the list of occurrencies. Then select the max key, get the value list and get the min (i.e., the sorted first value) and print it.


<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    string = argv[ 0 ]
    score = {}
    for i in range( 0, len( string ) - 1 ):
        needle = string[ i ] + string[ i + 1 ]
        counting = string.count( needle )
        if not counting in score:
            score[ counting ] = []

        if not needle in score[ counting ]:
            score[ counting ].append( needle )

    # get the highest key and extract the min value
    print( min( score[ max( score.keys() ) ] ) )

# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>
