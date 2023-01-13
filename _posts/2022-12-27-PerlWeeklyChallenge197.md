---
layout: post
title:  "Perl Weekly Challenge 197: Lists everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 197: Lists everywhere!

This post presents my solutions to the [Perl Weekly Challenge 197](https://perlweeklychallenge.org/blog/perl-weekly-challenge-197/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 197 - Task 1 - Raku](#task1)
- [PWC 197 - Task 2 - Raku](#task2)
- [PWC 197 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 197 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 197 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 197 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.


<a name="task1"></a>
## PWC 197 - Task 1 - Raku Implementation

Given a list of integers, keep their sorting and place all the zero numbers to the right side.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my ( @swapped );
    @swapped = | @list.grep( * !~~ 0 ), | @list.grep( * ~~ 0 );
    @swapped.join( ',' ).say;
}

```
<br/>
<br/>

A couple of `grep` to the rescue: I create a `@swapped` array that contains the flatten union of the lists of nonzero and zero numbers.

<a name="task2"></a>
## PWC 197 - Task 2 - Raku Implementation

Implement a *wiggle sort*: if the index of the array is even than the number must be less than the number at its right, otherwise it must be greater.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my @sorted = @list;
    my $done = False;

    #  list[0] < list[1] > list[2] < list[3]….
    while ( ! $done ) {
	$done = True;
	for 0 ..^ @sorted.elems - 1 -> $i {
	    if ( $i %% 2 ) {
		if ( @sorted[ $i ] >= @sorted[ $i + 1 ] ) {
		    # need to change
		    my $temp = @sorted[ $i ];
		    @sorted[ $i ] = @sorted[ $i + 1 ];
		    @sorted[ $i + 1 ] = $temp;
		    $done = False;
		}
	    }
	    else {
		if ( @sorted[ $i ] <= @sorted[ $i + 1 ] ) {
		    # need to change
		    my $temp = @sorted[ $i ];
		    @sorted[ $i ] = @sorted[ $i + 1 ];
		    @sorted[ $i + 1 ] = $temp;
		    $done = False;

		}
	    }

	}
    }

    @list.join( ',' ).say;
    @sorted.join(',').say;
}

```
<br/>
<br/>

I create a clone of the input array, named `@sorted` and then iterate over all the indexes, and if it is even I switch the elements if needed, and the same if the index is odd.
The `$done` flag is set to `True` whenere the cycle restarts, and if there are no changes the array is sorted.


<a name="task1plperl"></a>
## PWC 197 - Task 1 - PL/Perl Implementation

Same implementation of the Raku version, with a few more parentheses:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc197.task1_plperl( int[] )
RETURNS int[]
AS $CODE$
my ( $list ) = @_;
   my @sorted = ( grep( { $_ != 0 } $list->@* ),
                  grep( { $_ == 0 } $list->@* ) );
   return [ @sorted ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 197 - Task 2 - PL/Perl Implementation

Similar, but shorter, to the Raku implementation:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc197.task2_plperl( int[] )
RETURNS int[]
AS $CODE$
  my ( $array ) = @_;
  my $sorted = [ $array->@* ];
  my $need_swap = 1;

  while ( $need_swap ) {
    $need_swap = 0;
    for my $i ( 0 .. $sorted->@* - 1 ) {
      my $need_swap = ( ( $i % 2 == 0 ) && ( $sorted->[ $i ] >= $sorted->[ $i + 1 ] ) )
                      || ( ( $i % 2 != 0 ) && ( $sorted->[ $i ] <= $sorted->[ $i + 1 ] ) );

     if ( $need_swap ) {
        my $temp = $sorted->[ $i ];
        $sorted->[ $i ] = $sorted->[ $i + 1 ];
        $sorted->[ $i + 1 ] = $temp;
     }


   }
  }

  return $sorted;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The trick here is that the swapping is always the same, so I do a single test to see if there is the need for a change at the current iteration.


<a name="task1plpgsql"></a>
## PWC 197 - Task 1 - PL/PgSQL Implementation

More verbose, but similar to the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc197.task1_plpgsql( l int[] )
RETURNS int[]
AS $CODE$
DECLARE
   i int;
   v int[];
   zeros int := 0;
BEGIN
	FOREACH i IN ARRAY l LOOP
		IF i = 0 THEN
		   zeros := zeros + 1;
		   CONTINUE;
		END IF;

		v := v || i;
	END LOOP;

	WHILE zeros > 0 LOOP
	      v := v || 0;
	      zeros := zeros - 1;
	END LOOP;

	RETURN v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The problem is that PostgreSQL does not have a `grep` like function, so the implementation is more verbose than its counterparts in Perl or Raku.

<a name="task2plpgsql"></a>
## PWC 197 - Task 2 - PL/PgSQL Implementation

Same idea of the other implementations:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc197.task2_plpgsql( l int[] )
RETURNS int[]
AS $CODE$
DECLARE
    i int;
    v int[];
    need_change bool := false;
    t int;
BEGIN
    need_change := true;
    v := l;
    raise info 'array %', v;

    WHILE need_change LOOP
      need_change := false;
      FOR i IN 0 .. array_length( v, 1 ) - 1 LOOP

          IF i % 2 = 0 THEN
  	   IF v[ i ] <= v[ i + 1 ] THEN
  	      need_change := true;
  	   END IF;
  	ELSE
  	  IF v[i] >= v[ i + 1 ] THEN
  	     need_change := true;
  	  END IF;
  	END IF;


  	IF need_change THEN
  	  t := v[i];
  	  v[i] := v[i + 1];
  	  v[ i + 1 ]:= t;
  	END IF;
      END LOOP;

    END LOOP;



    RETURN v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Again, this is much more verbose than the other implementations, due to the lack of operators in `PL/PgSQL`.
There is an important thing to note: **in `PL/PgSQL` and in `SQL` in general, arrays begin at the index 1 so the first index is odd, not even!** This means that the test to check for changes must be swapped with regard to the Perl/Raku counterparts.
