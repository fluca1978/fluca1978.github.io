---
layout: post
title:  "Perl Weekly Challenge 212: jumping words and batching arrays"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 212: jumping words and batching arrays

This post presents my solutions to the [Perl Weekly Challenge 212](https://perlweeklychallenge.org/blog/perl-weekly-challenge-212/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 212 - Task 1 - Raku](#task1)
- [PWC 212 - Task 2 - Raku](#task2)
- [PWC 212 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 212 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 212 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 212 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 212 - Task 1 - Raku Implementation

The first task was about *jumping* a word, that is given a word, move every letter forward by the specified number of positions of a given array of integers, where every number corresponds to the offset of the next letter.

<br/>
<br/>
```raku
sub MAIN( *@args ) {
    my $word = @args[ 0 ];
    my @jumps = @args[ 1 .. * ];
    my @alphabet = 'a' .. 'z';
    my @new-world;
    my $index = 0;
    for $word.lc.comb {
		next if ! $_;
		next if ! @alphabet.grep( * ~~ $_ );
		my $jump = @jumps.shift;
		my $idx = ( $jump + @alphabet.first( $_, :k ) ) % @alphabet.elems;
		@new-world.push: @alphabet[ $idx ];
    }

    @new-world.join.say;
}

```
<br/>
<br/>

This is not a difficult task: I define an array that will contain the new word letters, and given the `@alphabet` I simply compute the next position (considering a modulo operation).



<a name="task2"></a>
## PWC 212 - Task 2 - Raku Implementation

Given an array of integers, divide it in set of sorted batches with the same number of elements.

<br/>
<br/>
```raku
sub MAIN( *@args ) {
    my $size = @args[ * - 1 ];
    my @list = @args[ 0 .. * - 2 ];

    # check if the size can be used to split the list
    '-1'.say and exit if ( @list.elems !%% $size );

    my $bag = Bag.new( @list ).Hash;

    my @batches;
    my @current;
    while ( @batches.elems != ( @list.elems / $size ) ) {

		my @available-keys = $bag.keys.grep( { $bag{ $_ } > 0 && ! @current.grep( $_ ) } );
		my $key = @available-keys.min;
		@current.push: $key;

		$bag{ $key } -= 1;
		$bag{ $key }:delete  if ( $bag{ $key } <= 0 );

		if ( @current.elems == $size ) {
		    @batches.push: [ @current ];
		    @current = ();
		}
    }

    @batches.join( "\n" ).say;
}

```
<br/>
<br/>

I'm sure there's a shorter way, but I cannot find it at the moment.
The idea is to keep track of every `@current` batch into an array of `@batches`, and to stop when I've used all the expected batches. I search for the minimum value that is still available, i.e., that has not been fully used into other batches, and that is not already inserted into the current batch. Then I decrease the availability of such value and, in the case, delete it from the `$bag` of values.
When the `@current` batch is ready, I append it and start it over.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 212 - Task 1 - PL/Perl Implementation

Same implementation as Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc212.task1_plperl( text, int[] )
RETURNS text
AS $CODE$
  my ( $string, $jumps ) = @_;
  my @alphabet = 'a' .. 'z';

  my $find_index = sub {
     my ( $letter ) = @_;
     for my $index ( 0 .. scalar( @alphabet ) ) {
     	 return $index if ( $alphabet[ $index ] eq $letter );
     }
  };

  my $offset = 0;
  my @word;
  for my $letter ( split //, $string ) {
      my $index = $find_index->( $letter );
      $index += $jumps->[ $offset++ ];
      $index %= @alphabet;
      push @word, $alphabet[ $index ];
  }

  return join( '', @word );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


The `find_index` routine is used to find where a letter is within the `@alphabet`; then the `$index` is added and modulo compared to get the new letter out from the alphateb. The resulted array of letters is then joined and returned.

<a name="task2plperl"></a>
## PWC 212 - Task 2 - PL/Perl Implementation

Same idea as the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc212.task2_plperl( int[], int )
RETURNS SETOF int[]
AS $CODE$
   my ( $list, $size ) = @_;
   return undef if ( scalar( $list->@* ) % $size != 0 );

   my $bag = {};
   $bag->{ $_ }++ for ( $list->@* );

   my $find_min_available = sub {
      my ( $bag, $array ) = @_;
      for my $k ( sort keys $bag->%* ) {
      	  if ( $bag->{ $k } > 0 && ! grep( {$_ == $k} $array->@* ) ) {
	     $bag->{ $k } -= 1;
	     return $k;
	  }
      }
   };

   my $done = 0;

   while ( $done < ( $list->@* / $size ) ) {
       my $current = [];
       while ( scalar( $current->@* ) != $size ) {
       	 my $value = $find_min_available->( $bag, $current );
	     return undef if ! $value;
	     push $current->@*, $value;
       }

      return_next( $current );
      $done++;
   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `find_min_available` function does pretty much all the magic selecting the minimum value available and adjusting the availability, also checking if the value has been already inserted into the `$current` batch.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 212 - Task 1 - PL/PgSQL Implementation

Here I use a table to represent the alphabet.


<br/>
<br/>
```sql
CREATE TABLE IF NOT EXISTS pwc212.alphabet
(
  l char
  , n int
  , PRIMARY KEY( l )
);

TRUNCATE pwc212.alphabet;
INSERT INTO pwc212.alphabet
SELECT l, row_number() over ()
FROM regexp_split_to_table( 'abcdefghijklmnopqrstuvwxyz', '' ) l;



CREATE OR REPLACE FUNCTION
pwc212.task1_plpgsql( s text, jumps int[] )
RETURNS text
AS $CODE$
DECLARE
	letter text;
	word text;
	idx int;
	off int := 0;
	alphabet_size int;
BEGIN

	SELECT count(*)
	INTO alphabet_size
	FROM pwc212.alphabet;

	word := '';

	FOR letter IN SELECT * FROM regexp_split_to_table( s, '' )  LOOP
	    SELECT n
	    INTO  idx
	    FROM pwc212.alphabet
	    WHERE l = letter;


	    SELECT mod( i + idx, alphabet_size )
	    INTO idx
	    FROM unnest( jumps ) i
	    LIMIT 1 	    OFFSET off;
	    off := off + 1;

	    SELECT l
	    INTO letter
	    FROM pwc212.alphabet
	    WHERE n = idx;

	    word := word || letter;

	END LOOP;

	RETURN word;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The idea is then to select the index of the current letter out of the alphabet table, then compute the next index and get the new letter out of the same table. The trick of using `LIMIT 1 OFFSET off` is to select a single row for the index to add out of the `jumps` array.


<a name="task2plpgsql"></a>
## PWC 212 - Task 2 - PL/PgSQL Implementation

Same idea as before, but the *bag* is represented via a temporary table that is initialized with the counting of the elements out of the `unnest`ing of the given array.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc212.task2_plpgsql( a int[], s int)
RETURNS SETOF int[]
AS $CODE$
DECLARE
	current int[];
	done    int := 0;
	next_value int;
BEGIN

	-- check if the array can be divided into batches
	IF mod( array_length( a, 1 ), s ) <> 0 THEN
	   RETURN;
	END IF;

	CREATE TEMPORARY TABLE IF NOT EXISTS bag( v int, c int default 1 );
	TRUNCATE TABLE bag;
	INSERT INTO bag
	SELECT v, count(*)
	FROM unnest( a ) v
	GROUP BY v;


	WHILE done < ( array_length( a, 1 ) / s ) LOOP
	      current = array[]::int[];

	      WHILE array_length( current, 1 ) IS NULL OR array_length( current, 1 ) < s LOOP
	      	    SELECT min( v )
		    INTO next_value
		    FROM   bag
		    WHERE  c > 0
		    AND   v NOT IN ( SELECT * FROM unnest( current ) );

		    UPDATE bag
		    SET c = c - 1
		    WHERE v = next_value;

		    current := array_append( current, next_value );
	      END LOOP;

	      done := done + 1;
	      RETURN NEXT current;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

While preparing the current batch, I select the minimum value still available ensuring also it does not appear into the curent batch by means of a `NOT IN` subquery. Then I deselect the availability of such value and append it into the `current` batch.
