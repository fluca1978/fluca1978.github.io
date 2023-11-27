---
layout: post
title:  "Perl Weekly Challenge 245: Hashes and Joins"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 245: Hashes and Joins

This post presents my solutions to the [Perl Weekly Challenge 245](https://perlweeklychallenge.org/blog/perl-weekly-challenge-245/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 245 - Task 1 - Raku](#task1)
- [PWC 245 - Task 1 - Raku (another implementation)](#task1b)
- [PWC 245 - Task 2 - Raku](#task2)
- [PWC 245 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 245 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 245 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 245 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 245 - Task 1 in Python](#task1python)
- [PWC 245 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 245 - Task 1 - Raku Implementation

The first task was about sorting a first hash of language names on the basis of a second array of language popularities.

I decided to remap the languages into an hash, keyed by the popularity, and then reprint it after having sorted the keys.

<br/>
<br/>
```raku
sub MAIN( :@langs where { @langs.elems == @langs.grep( * ~~ Str ).elems }
	  , :@popularity where { @popularity.elems == @langs.elems == @popularity.grep( * ~~ Int ).elems } ) {

    my %sorted;
    %sorted{ @popularity[ $_ ] } = @langs[ $_ ] for 0 ..^ @langs.elems;

    my @output;
    @output.push: %sorted{ $_ } for %sorted.keys.sort( { $^b <=> $^a } );
    @output.join( ', ' ).say;

}

```
<br/>
<br/>


<a name="task1b"></a>
## PWC 245 - Task 1 - Raku Implementation (another approach)

It is possible to use another approach to solve the problem:

<br/>
<br/>
```raku
sub MAIN( :@langs where { @langs.elems == @langs.grep( * ~~ Str ).elems }
	  , :@popularity where { @popularity.elems == @langs.elems == @popularity.grep( * ~~ Int ).elems } ) {

    ( @langs [Z] @popularity ).sort( { $^b[1] <=> $^a[1] } ).map( *[ 0 ] ).join( ',' ).say;
}

```
<br/>
<br/>

What happens is:
- `[Z]` merges the arrays on a same index basis. The result is a set of arrays made by a language and its popularity;
- `sort` does sort every element using the second (i.e., indexed as `1`) element, that is the popularity;
- `map` returns only the first element of each item, that is the language;
- `join` and `say` does the printing.



<a name="task2"></a>
## PWC 245 - Task 2 - Raku Implementation

The second task was about finding out the max value of the concatenation of a given array of digits, assuming the value must be a multiple of `3`.


<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems
					 && @nums.grep( 0 <= * <= 9 ).elems } ) {

    my $largest = -1;
    for @nums.permutations {
		my $value = $_.join.Int;
		next if $value !%% 3;
		$largest = $value if ( $value > $largest );
    }

    $largest.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 245 - Task 1 - PL/Perl Implementation

Same implementation as in Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc245.task1_plperl( text[], int[] )
RETURNS SETOF text
AS $CODE$
   my ( $langs, $popularity ) = @_;

  die "Not same length arrays!" if ( $langs->@* != $popularity->@* );

   my $sorting = {};

    for my $index ( 0 .. $popularity->@* - 1 ) {
        $sorting->{ $popularity->[ $index ] } = $langs->[ $index ];
    }

    for ( sort { $b <=> $a } $popularity->@* ) {
        return_next( $sorting->{ $_ } );
    }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 245 - Task 2 - PL/Perl Implementation

This time I had to use `Algorithm::Combinatorics` to get a quick access to a `permutations` method.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc245.task2_plperl( int[] )
RETURNS int
AS $CODE$
   use Algorithm::Combinatorics qw/ permutations /;
   my ( $digits ) = @_;

   die "Digits must be between 0 and 9" if ( grep( { $_ > 9 || $_ < 0 } $digits->@* ) );

   my $result = -1;
   for my $k ( 0 .. $digits->@* ) {
     my $permutations = permutations( \ $digits->@*, $k );
     while ( my $iter = $permutations->next ) {
     	   my $value = join('', $iter->@* );
		   next if ( $value % 3 != 0 );
		   $result = $value if ( $value > $result );
     }
   }

   return $result;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 245 - Task 1 - PL/PgSQL Implementation

This task is quite simple: I can `unnest` the arrays, joining them, sorting by popularity and then filtering to get distinct values. In other words, it can be done with a single query.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc245.task1_plpgsql( langs text[], popularity int[] )
RETURNS SETOF text
AS $CODE$

	WITH sorting AS (
		SELECT l
		FROM unnest( langs ) l
		, unnest( popularity ) p
		ORDER BY p DESC
	)
	SELECT distinct( l )
	FROM sorting;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 245 - Task 2 - PL/PgSQL Implementation


Doing permutations in SQL is quite hard, and requires a recursive CTE. Therefore, I cheat here and use the Perl implementation!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc245.task2_plpgsql( digits int[] )
RETURNS int
AS $CODE$
   SELECT pwc245.task2_plperl( digits );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 245 - Task 1 - Python Implementation

The idea is the same as in Raku, but the arrays are two command line strings with a pipe separator.


<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    langs = argv[ 0 ].split( "|" )
    popularity = list( map( int, argv[1].split( "|" ) ) )

    sorting = {}
    for i in range( 0, len( popularity ) ):
        sorting[ langs[ i ] ] = popularity[ i ]


    for v in sorted( sorting.items(), key=lambda x: x[1], reverse=True ):
        print( v[0] )



# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 245 - Task 2 - Python Implementation

The idea is the same as in Raku, but to get the permutations I need an external library.


<br/>
<br/>
```python
import sys
from itertools import permutations

# task implementation
def main( argv ):
    digits = list( map( int, argv ) )

    result = -1

    for p in permutations( digits ):
        value = int( "".join( map( str, p ) ) )

        if value % 3 != 0:
            continue

        if value > result:
            result = value

    print( result )



# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>
