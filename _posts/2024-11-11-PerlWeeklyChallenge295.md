---
layout: post
title:  "Perl Weekly Challenge 295: loops and substitutions"
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

# Perl Weekly Challenge 295: loops and substitutions

This post presents my solutions to the [Perl Weekly Challenge 295](https://perlweeklychallenge.org/blog/perl-weekly-challenge-295/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 295 - Task 1 - Raku](#task1)
- [PWC 295 - Task 2 - Raku](#task2)
- [PWC 295 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 295 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 295 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 295 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 295 - Task 1 in PL/Java](#task1pljava)
- [PWC 295 - Task 2 in PL/Java](#task2pljava)
- [PWC 295 - Task 1 in Python](#task1python)
- [PWC 295 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 295 - Task 1 - Raku Implementation

The first task was to find out if, given an array of words, a given sentence could be broken into all the words specified.

<br/>
<br/>
```raku
sub MAIN( Str $string is copy, *@words ) {
    for @words -> $current-word {
		last if ( $string !~~ /$current-word/ );
		$string .= subst( $current-word, '', :g );
    }

    'True'.say if ! $string;
    'False'.say if $string;
}

```
<br/>
<br/>

The idea is to iterate over the given `@words` and see the `$string` matches the given word. If it does not, then we can stop looping, otherwise remove the given word and continue over.
In the end, if the sentence has no more chars, then we have succeeded, otherwise there are parts that cannot be tokenized.



<a name="task2"></a>
## PWC 295 - Task 2 - Raku Implementation

I admit I'm not sure I've understood this second task, however, given an array of integers, compute the min *jump path* to the end.
As far as I understand, we start at the very first index, get the jump value as the value of the cell and move forward to the index pointed by the jump, and continue unless we reach the last element. Or, start from the leftmost index, move by one position and jump from the second, and on on.


<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems } ) {

    my ( $position ) = 0;
    my $min-jumps = @numbers.elems + 1;
    for 0 ..^ @numbers.elems - 1 -> $index {
		$position = $index;
		my $jump-counter = $index;

	    while ( $position < @numbers.elems - 1 ) {
			my $jump = @numbers[ $position ];
			last if ! $jump;
			$position += $jump;
			$jump-counter++;
		}

	   $min-jumps = $jump-counter if $jump-counter < $min-jumps && $position == @numbers.elems - 1;
    }

    $min-jumps.say and exit if ( $min-jumps < @numbers.elems );
    '1'.say;

}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 295 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, using regular expression.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc295.task1_plperl( text, text[] )
RETURNS bool
AS $CODE$

   my ( $sentence, $words ) = @_;

   for ( $words->@* ) {
       last if $sentence !~ /$_/;
       $sentence =~ s/$_//g;
   }

   return $sentence ? 0 : 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 295 - Task 2 - PL/Perl Implementation

The exact same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc295.task2_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $numbers ) = @_;
   my $position;
   my ($min) = $numbers->@* + 1;

   for my $index ( 0 .. $numbers->@* - 2 ) {

       $position = $index;
       my $counter = $index;

       while ( $position < $numbers->@* - 1 ) {
	     my $jump = $numbers->@[ $position ];
	     last if ! $jump;
	     $position += $jump;
	     $counter++;
       }

       $min = $counter if ( $counter < $min && $position == $numbers->@* - 1 );
   }

   return $min if ( $min < $numbers->@* );
   return -1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 295 - Task 1 - PL/PgSQL Implementation


Use regular expressions to match and substitute.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc295.task1_plpgsql( sentence text, words text[] )
RETURNS bool
AS $CODE$
DECLARE
	word text;
BEGIN

	FOREACH word IN ARRAY words LOOP
		IF NOT sentence ~ word THEN
		   RETURN false;
		END IF;

		SELECT regexp_replace( sentence, word, '', 'g' )
		INTO sentence;
	END LOOP;

	RETURN length( sentence ) = 0;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 295 - Task 2 - PL/PgSQL Implementation

Here I cheat, and call the PL/Perl implementation.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc295.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
   SELECT pwc295.task2_plperl( nums );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 295 - Task 1 - PostgreSQL PL/Java Implementation

Same implementation as in Raku.


<br/>
<br/>
```java
public static final boolean task1_pljava( String sentence, String[] words ) throws SQLException {
	logger.log( Level.INFO, "Entering pwc295.task1_pljava" );

	for ( String word : words ) {
	    if ( ! sentence.contains( word ) )
		return false;

	    sentence = sentence.replaceAll( word, "" );
	}

	return sentence.length() == 0;
}
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 295 - Task 2 - PostgreSQL PL/Java Implementation


Same implementation as in Raku.


<br/>
<br/>
```java
   public static final int task2_pljava( int[] numbers ) throws SQLException {
	logger.log( Level.INFO, "Entering pwc295.task2_pljava" );

	int position = 0;
	int min      = numbers.length + 2;

	for ( int index = 0; index < numbers.length - 1; index++ ) {
	    position = index;
	    int counter = index;

	    while ( position < numbers.length - 1 ) {
			int jump = numbers[ position ];
			if ( jump == 0 )
			    break;

			position += jump;
			counter++;
	    }

	    if ( counter < min && position == numbers.length - 1)
			min = counter;
	}


	if ( min < numbers.length )
	    return min;
	else
	    return -1;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 295 - Task 1 - Python Implementation

Here I don't have to use regular expressions.


<br/>
<br/>
```python
# task implementation
# the return value will be printed
def task_1( args ):
    sentence = args[ 0 ]
    words    = args[ 1: ]

    for word in words:
        if not word in sentence:
            return False

        sentence = sentence.replace( word, '' )

    return len( sentence ) == 0


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 295 - Task 2 - Python Implementation


Same implementation, copied from Raku.


<br/>
<br/>
```python
# task implementation
# the return value will be printed
def task_2( args ):
    numbers = list( map( int, args ) )

    position = 0
    jumps = len( numbers ) + 2

    for index in range( 0, len( numbers ) - 2 ):
        position = index
        counter  = index


        while position < len( numbers ) - 1 :
            jump = numbers[ position ]
            if jump == 0:
                break

            position += jump
            counter += 1



        if counter < jumps and position == ( len( numbers ) - 1 ):
            jumps = counter

    if jumps < len( numbers ) - 1:
        return jumps
    else:
        return -1



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
