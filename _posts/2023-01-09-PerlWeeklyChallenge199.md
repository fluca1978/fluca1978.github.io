---
layout: post
title:  "Perl Weekly Challenge 199: Nested Loops Everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 199: Nested Loops Everywhere!

This post presents my solutions to the [Perl Weekly Challenge 199](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0199/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 199 - Task 1 - Raku](#task1)
- [PWC 199 - Task 2 - Raku](#task2)
- [PWC 199 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 199 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 199 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 199 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.


<a name="task1"></a>
## PWC 199 - Task 1 - Raku Implementation

The first task was about finding out *good numbers* withing a list of integers, where a good pair of numbers is a pair where `a < b` being `a` coming first than `b` in the list.
<br/>
I decided to implement this with a nested loop:

<br/>
<br/>
```raku
sub MAIN( Bool :$verbose = False,
	*@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my @pair-indexes;
    for 0 ..^ @list.elems -> $i {
	for $i ^..^ @list.elems -> $j {
	    next if $i == $j;
	    @pair-indexes.push: [ $i, $j, @list[ $i ], @list[ $j ] ] if @list[ $i ] == @list[ $j ];
	}
    }

    @pair-indexes.elems.say;
    @pair-indexes.map( { 'Idexes ' ~ $_[0] ~ ',' ~ $_[1] ~ ' refer to equal elements ' ~ $_[2] ~ ',' ~ $_[3] } ).join( "\n" ).say if $verbose;
}

```
<br/>
<br/>

In the case of the `$verbose` flag, there is a mapping of the elements into a stringified version so that I can see what the program is choosing.


<a name="task2"></a>
## PWC 199 - Task 2 - Raku Implementation

The second task was about finding out *good triplets* among a list of integers, where a triplet is consider good if the numbers are ordered by left to right within the list and their absolute difference is less than (or equal) a given input value.
Again, I implemented this with a triple nested loop:

<br/>
<br/>
```raku
sub MAIN(  Int $x,
	   Int $y,
	   Int $z,
	   Bool :$verbose = False,
          *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {

    my @triplets;
    for 0 ..^ @list.elems -> $i {
	for $i ^..^@list.elems -> $j {
	    for $j ^..^ @list.elems -> $k {
		@triplets.push: [ $i, $j, $k, @list[ $i ], @list[ $j ], @list[ $k ] ]
				    if ( ( @list[ $i ] - @list[ $j ] ).abs <= $x
					&& ( @list[ $j ] - @list[ $k ] ).abs <= $y
					&& ( @list[ $i ] - @list[ $k ] ).abs <= $z );

	    }
	}
    }

    @triplets.elems.say;
    @triplets.map( { "Indexes $_[0], $_[1], $_[2] are good ($_[3], $_[4], $_[5])" } ).join( "\n" ).say;
}

```
<br/>
<br/>



<a name="task1plperl"></a>
## PWC 199 - Task 1 - PL/Perl Implementation

Pure translation of the Raku implementation into Perl:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc199.task1_plperl( int[] )
RETURNS int
AS $CODE$

  my ( $list ) = @_;
  my @pairs;

  for my $i ( 0 .. $list->@* ) {
   for my $j ( $i .. $list->@* ) {
     next if $i == $j;
     push @pairs, [ $i, $j ] if ( $list->[ $i ] == $list->[ $j ] );
   }
 }

 return scalar @pairs;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 199 - Task 2 - PL/Perl Implementation

Another translation from Raku to Perl:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc199.task2_plperl( int, int, int, int[] )
RETURNS int
AS $CODE$
  my ( $x, $y, $z, $list ) = @_;
  my @triplets;

  for my $i ( 0 .. $list->@* ) {
    for my $j ( $i + 1 .. $list->@* - 1 ) {
      for my $k ( $j + 1 .. $list->@* - 2 ) {

        push @triplets, [ $i, $j, $k ] if ( abs( $list->[ $i ] - $list->[ $j ] ) <= $x
	                                 && abs( $list->[ $j ] - $list->[ $k ] ) <= $y
					 && abs( $list->[ $i ] - $list->[ $k ] ) <= $z );
      }
    }
  }


  return scalar @triplets;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task1plpgsql"></a>
## PWC 199 - Task 1 - PL/PgSQL Implementation

Similar to the PL/Perl implementation, but this time I do not keep track of the pairs themselves, rather I simply count them:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc199.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
	i int;
	j int;
	c int := 0;
BEGIN
	FOR i IN 1 .. array_length( l, 1 ) LOOP
	    FOR j IN i .. array_length( l , 1 ) LOOP
	    	If i = j THEN
		   CONTINUE;
		END IF;

		IF l[i] = l[j] THEN
		   c := c + 1;
		END IF;
	    END LOOP;
	END LOOP;

	RETURN c;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 199 - Task 2 - PL/PgSQL Implementation

Similar to the PL/Perl implementation, but again here I do not take into account the triplets themselves, rather I only do count them:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc199.task2_plpgsql( x int, y int, z int, l int[] )
RETURNS int
AS $CODE$
DECLARE
	i int;
	j int;
	k int;
	c int := 0;
BEGIN
	FOR i IN 1 .. array_length( l, 1 ) LOOP
	    FOR j IN ( i + 1 ) .. array_length( l, 1 ) LOOP
	    	FOR k IN ( j + 1 ) .. array_length( l, 1 ) LOOP
		    IF abs( l[i] - l[j] ) <= x AND abs( l[j] - l[k] ) <= y AND abs( l[i] - l[k] ) <= z THEN
		       c := c + 1;
		    END IF;
		END LOOP;
	    END LOOP;
	END LOOP;

	RETURN c;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
