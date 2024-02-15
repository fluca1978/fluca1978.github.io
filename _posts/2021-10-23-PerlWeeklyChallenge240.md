---
layout: post
title:  "Perl Weekly Challenge 240: case folding and the jumping array"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 240: case folding and the jumping array

This post presents my solutions to the [Perl Weekly Challenge 240](https://perlweeklychallenge.org/blog/perl-weekly-challenge-240/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 240 - Task 1 - Raku](#task1)
- [PWC 240 - Task 2 - Raku](#task2)
- [PWC 240 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 240 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 240 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 240 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 240 - Task 1 in Python](#task1python)
- [PWC 240 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 240 - Task 1 - Raku Implementation

The first task was about finding out if a given string is an acronym of a list of strings, that means if the first letter of every string is in the array of letters of the given acronyms, without any regard to the case, and in the right order.
A first approach can use a loop:

<br/>
<br/>
```raku
sub MAIN( $check-string, *@strings ) {
    # the check string must contain the same
    # exact number of chars of the number of strings
    'False'.say and exit if ( $check-string.comb.elems != @strings.elems );

     # check all the letters
     my @letters = $check-string.comb;
     for 0 ..^ @strings.elems {
     	'False'.say and exit if ( @letters[ $_ ].fc ne ( @strings[ $_ ].comb )[ 0 ].fc );
     }

     'True'.say;
}
```
<br/>
<br/>

The idea is to check immediatly if the number of letters match the number of strings, otherwise there is no point in doing a deep comparison.

**Clearly, the comparison has to be done using the `fc` (*fold case*) operator in order to properly do a case-insensitive comparison**.

Then, there could be a shorter implementation:

<br/>
<br/>
```perl6
sub MAIN( $check-string, *@strings ) {
    # the check string must contain the same
    # exact number of chars of the number of strings
    'False'.say and exit if ( $check-string.comb.elems != @strings.elems );

    my @check-letters = $check-string.comb.map( *.fc );
    my @letters = @strings.map( ( *.comb )[ 0 ].fc );
    'True'.say and exit if ( @letters ~~ @check-letters );

    'False'.say;
}

```
<br/>
<br/>

This exploits the *smart matching* operator that does the right thing when testing arrays.

<a name="task2"></a>
## PWC 240 - Task 2 - Raku Implementation

This task was about producing an array out of aginve input array, jumping to cells that are the result of a given array cell.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems } ) {
    my @new;
    @new[ $_ ] = @numbers[ @numbers[ $_ ] ] for 0 ..^ @numbers.elems;
    @new.join( ', ' ).say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 240 - Task 1 - PL/Perl Implementation

Pretty much the same implementation of Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc240.task1_plperl( text, text[] )
RETURNS boolean
AS $CODE$
  my ( $check, $strings ) = @_;
  my @check_letters = map { $_ } split( //, $check );
  my @letters = map {  ( split //, $_  )[ 0 ] } $strings->@*;

  return 0 if ( @check_letters != @letters );

  for ( 0 .. @check_letters ) {
      return 0 if ( CORE::fc( $check_letters[ $_ ] ) ne CORE::fc( $letters[ $_ ] ) );
  }

  return 1;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


Note that I used the `CORE::fc` in order to avoid using an explicit `use` that would require to turn the function into untrusted language.


<a name="task2plperl"></a>
## PWC 240 - Task 2 - PL/Perl Implementation

Really simple to implement in PL/Perl, thanks to the `return_next` function that will append a new result to the output:


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc240.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $numbers ) = @_;

   for ( 0 .. $numbers->@* ) {
       return_next( $numbers->[ $numbers->[ $_ ] ] );
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 240 - Task 1 - PL/PgSQL Implementation

I used a nested loop and the internal function `regexp_split_to_array` to get the letters out of a string:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc240.task1_plpgsql( c text, s text[] )
RETURNS boolean
AS $CODE$
DECLARE
	check_letters text[];
	current_index int;
	current_letter text;
BEGIN
	check_letters := regexp_split_to_array( c, '' );

	IF array_length( check_letters, 1 ) <> array_length( s, 1 ) THEN
	   RETURN FALSE;
	END IF;

	FOR current_index IN 1 .. array_length( check_letters, 1 ) LOOP
	    current_letter := ( regexp_split_to_array( s[ current_index ], '' ) )[ 1 ];
	    IF lower( current_letter ) <> lower( check_letters[ current_index ] ) THEN
	       RETURN FALSE;
	    END IF;
	END LOOP;

	RETURN TRUE;
END
$CODE$

```
<br/>
<br/>

Please consider that in SQL the numbering starts from `1`!


<a name="task2plpgsql"></a>
## PWC 240 - Task 2 - PL/PgSQL Implementation

Similar to the PL/Perl implementation:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc240.task2_plpgsql( n int[] )
RETURNS SETOF INT
AS $CODE$
DECLARE
	current_index int;
	position  int;
BEGIN
	FOR current_index IN 1 .. array_length( n, 1 ) LOOP
	    position := n[ current_index ];
	    IF position = 0 THEN
	       position := 1;
	    END IF;
	    RETURN NEXT n[ position ];
	END LOOP;

	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I need to use an intermediate variable `position` because, since in SQL numbering starts from `1`, chances are that the index position is a zero and need to be converted appropriately.


# Python Implementations

<a name="task1python"></a>
## PWC 240 - Task 1 - Python Implementation

Same implementation as above, the function prints and returns the corresponding value.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    letters = list( map( lambda x: x[ 0 ].casefold(), argv[1:] ) )
    check_chars = list( argv[ 0 ] )

    if len( check_chars ) != len( letters ):
        print( 'False' )
        return False

    for index in range( len( argv[ 0 ] ) ):
        if argv[ 0 ][ index ].casefold() != letters[ index ]:
            print( 'False' )
            return False

    print( 'True' )
    return True


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )


```
<br/>
<br/>

I use the function `casefold` to get the case folding, and a lambda expression to work with `map` in a Perl-ish way and get out the first char of every string.

<a name="task2python"></a>
## PWC 240 - Task 2 - Python Implementation

In a similar way to the previous implementations, this one creates an array and fills with the *jumps* into the old one:

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    new_array = []
    for index in range( len( argv ) ):
        new_array.insert( index, argv[ int( argv[ index ] ) ] )

    print( ','.join( new_array ) )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>
