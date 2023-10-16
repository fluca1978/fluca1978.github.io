---
layout: post
title:  "Perl Weekly Challenge 239: arrays, grep and nested loops"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 239: arrays, grep and nested loops

This post presents my solutions to the [Perl Weekly Challenge 239](https://perlweeklychallenge.org/blog/perl-weekly-challenge-239/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 239 - Task 1 - Raku](#task1)
- [PWC 239 - Task 2 - Raku](#task2)
- [PWC 239 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 239 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 239 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 239 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 239 - Task 1 in Python](#task1python)
- [PWC 239 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 239 - Task 1 - Raku Implementation

The first task was about finding out if two given arrays, once contatenated, result in the same string.
This is really easy thanks to the `join` operator:

<br/>
<br/>
```raku
sub MAIN( :@words1, :@words2 ) {
    'true'.say and exit if ( @words1.join ~~ @words2.join );
    'false'.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 239 - Task 2 - Raku Implementation

The second task was to find out which words of a given array contains only allowed letters, passed as a string in the parameter list.

<br/>
<br/>
```raku
sub MAIN( Str $allowed, *@words ) {
    my @allowed;
    for @words -> $current-word {
		my $found = True;

		for $current-word.comb -> $current-char {
		    $found = False and last if ( ! $allowed.comb.grep( * ~~ $current-char ) );
		}

		@allowed.push: $current-word if ( $found );
    }

    @allowed.join( ', ' ).say;
}

```
<br/>
<br/>

I used the `$allowed` string as first argument, so that the remaining can be a slurpy array.
Then I loop on every word in the array, and see if any char (obtained by `comb`) `grep`s with the allowed string of chars, otherwise the word cannot be passed.
If the string passes the check, then I add it to the `@allowed` array and print it.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 239 - Task 1 - PL/Perl Implementation


Pretty much the very same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc239.task1_plperl( text[], text[] )
RETURNS bool
AS $CODE$
   my ( $words1, $words2 ) = @_;

   return 1 if ( join('', $words1->@*) eq join( '', $words2->@* ) );
   return 0;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 239 - Task 2 - PL/Perl Implementation


The same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc239.task2_plperl( text, text[] )
RETURNS SETOF text
AS $CODE$
   my ( $allowed, $words ) = @_;

   for my $word ( $words->@* ) {
       my $found = 1;
       for my $char ( split( //, $word ) ) {
       	   $found = 0 and last if ( ! grep( { $_ eq $char } split( //, $allowed ) ) );
       }

       return_next( $word ) if ( $found );
   }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

This time I don't have to use an intermediate array, rather I return at every time a good word by means of `return_next`.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 239 - Task 1 - PL/PgSQL Implementation

This can be solved with a single query, since PostgreSQL provides an aggregation function:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc239.task1_plpgsql( w1 text[], w2 text[] )
RETURNS boolean
AS $CODE$

SELECT
	array_to_string( w1, '' ) = array_to_string( w2, '' );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 239 - Task 2 - PL/PgSQL Implementation


This is similar to the PL/Perl implementation, but instead of `grep` I need to use the PostgreSQL function `regexp_like` to see if a letter is contained in the allowed string.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc239.task2_plpgsql( allowed text, words text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
	current_word text;
	current_char text;
	ok boolean;
BEGIN
	FOREACH current_word IN ARRAY words LOOP
		ok := true;

		FOREACH current_char IN ARRAY regexp_split_to_array( current_word, '' ) LOOP
			IF NOT regexp_like( allowed, current_char ) THEN
			   ok := false;
			   EXIT;
		       END IF;
		END LOOP; -- foreach

		IF ok THEN
		   RETURN NEXT current_word;
		END IF;
	END LOOP;  -- foreach external

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 239 - Task 1 - Python Implementation

Here I use a number to indicate where the arrays begins, so for instance the arguments are: `2 'ab 'c' 3 'a' b' 'c'` where the numbers indicate the size of the array, therefore the first one is made by two elements and the second array is made by three elements. With this trick in mind, I can split the `argv` array into the two distinct arrays. Then it is a matter of testing the `join`ess of the two.


<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    words1 = argv[ 1 : int( argv[ 0 ] ) + 1 ]
    words2 = argv[ int( argv[ 0 ] ) + 2 : ]

    if ''.join( words1 ) == ''.join( words2 ):
        print( "true" )
    else:
        print( "false" )




# invoke the main without the command itself
# $ python3 ch-1.py 2 'ab' 'c' 2 'a' 'bc'
# true
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 239 - Task 2 - Python Implementation


In this implementation I needed a more depth nested loops to test every char against every allowed char.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    allowed = argv[ 0 ]

    for current_word in argv[ 1: ] :
        found = True
        for current_char in list( current_word ):
            found_this_char = False

            for allowed_char in list( allowed ):
                if current_char == allowed_char:
                    found_this_char = True
                    break

            if not found_this_char:
                found = False
                break

        if found:
            print( current_word )



# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>

Again, the first word in the parameter list is the allowed word.
Then I iterate on all the given words, and on all the characters of the word, and then on the allowed chars. This is where something like `grep` would greatly improve the code!
