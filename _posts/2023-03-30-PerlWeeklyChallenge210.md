---
layout: post
title:  "Perl Weekly Challenge 210: I didn't get very well!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 210: I didn't get very well!

This post presents my solutions to the [Perl Weekly Challenge 210](https://perlweeklychallenge.org/blog/perl-weekly-challenge-210/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 210 - Task 1 - Raku](#task1)
- [PWC 210 - Task 2 - Raku](#task2)
- [PWC 210 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 210 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 210 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 210 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 210 - Task 1 - Raku Implementation

The first task was about implementing a *kill and ring* game, where an input array of integers is trimmed in steps and every number cut off increments the total score.

<br/>
<br/>
```raku
sub MAIN( *@list is copy where { @list.grep( * ~~ Int).elems == @list.elems } ) {

    my @removed;

    for 2 .. @list.max {
	  next if @removed.grep( * == $_ );
	  next if @removed.grep( * == ( $_ + 1 ) );
	  next if @removed.grep( * == ( $_ - 1 ) );
	  @removed.push: @list.grep( * == $_ ), @list.grep( * == ( $_ + 1 ) ), @list.grep( * == ( $_ - 1 ) );
    }

    @removed.sum.say;
}

```
<br/>
<br/>

My understanding of the problem, that could have been solved by means of a simple *sum* over all the elements of the array, is that at every step I pick a number, and see if it has a plus/minus value, then I can kill the tuple made by the number and its plus/minus one counterparts. The sum of the erased number accumulates into the score.

In order to do the above, I pushed all the lists into the `@removed` one, since I knew that after all I'm going to erase all of them. Then, iut is only a matter of summing the values.


<a name="task2"></a>
## PWC 210 - Task 2 - Raku Implementation

The second task was about moving numbers from left to right or viceversa in an array: if the number is negative it moves one place left, otherwise it moves one place right. When a number moves, it *collides* with another one and that with the minimum absolute value *explodes*, that means it is erased from the array.

<br/>
<br/>
```raku
sub MAIN( *@list is copy where { @list.grep( * ~~ Int ).elems == @list.elems } ) {

    my $move = True;

    while ( $move ) {
    	$move = False;
		for 0 ..^ @list.elems - 1 {

		    my ( $left, $right ) = @list[ $_ ], @list[ $_ + ( @list[ $_ ] > 0 ?? 1 !! -1 ) ];
		    next if ( ! $left || ! $right );
		    next if ( $left > 0 && $right > 0 );
		    next if ( $left < 0 && $right < 0 );

		    $move = True;
		    @list[ $_ + ( @list[ $_ ] > 0 ?? 1 !! -1 ) ] = Nil if ( $left.abs >= $right.abs );
		    @list[ $_ ]     = Nil if ( $left.abs <= $right.abs );

		}

	    @list = @list.grep( * ~~ Int );
    }

    @list.grep( * ~~ Int ).join( ', ' ).say;
}
```
<br/>
<br/>


I implemented as follows:
- `$left` and `$right` are the two numbers I'm going to analyze at every step;
- if the numbers have both the same sign, then no move happens, so continue to the next iteration;
- otherwise a `$move` is necessary, and I set to `Nil` the number that has to explode. Since if the numbers are the same, in absolute value, both must explode, I test with less-or-equal and greater-or-equal.

In the end, I take care only of non-`Nil` values.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 210 - Task 1 - PL/Perl Implementation

Same implementation as in Raku, a little more verbose because I need some other *tools* to achieve the same result:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc210.task1_plperl( int[] )
RETURNS int
AS $CODE$
  my ( $list ) = @_;
  my $max = 0;
  my @removed;

  for ( $list->@* ) {
     $max = $_ if( $max < $_ );
  }

  for my $index ( 2 .. $max ) {
     next if ( grep { $_ == $index } @removed );
     next if ( grep { $_ == ( $index + 1 ) } @removed );
     next if ( grep { $_ == ( $index - 1 ) } @removed );

     push @removed, ( grep( { $_ == $index } $list->@* ),
                      grep( { $_ == ( $index + 1) } $list->@* ),
		      grep( { $_ == ( $index - 1) } $list->@* ) );
  }

  my $sum = 0;
  $sum += $_ for ( @removed );
  return $sum;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 210 - Task 2 - PL/Perl Implementation


Same approach as in Raku:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc210.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
  my ( $list ) = @_;
  my $move = 1;

  while ( $move ) {
     $move = 0;
     for my $index ( 0 .. scalar( $list->@* ) - 1 ) {
     	 my $offset = $list->[ $index ] > 0 ? 1 : -1;

         my ( $left, $right ) = ( $list->[ $index ], $list->[ $index + $offset ] );
	 next if ( ! $left || ! $right );
	 next if ( $left > 0 && $right > 0 );
	 next if ( $left < 0 && $right < 0 );


	 $move++;
	 $right *= ( $right < 0 ? -1 : 1 );
	 $left *= ( $left < 0 ? -1 : 1 );

	 $list->[ $index ] = 0 if ( $left <= $right );
	 $list->[ $index + $offset ] = 0 if ( $left >= $right );
     }

     $list = [ grep { $_ != 0 } $list->@* ];
  }

  for ( $list->@* ) {

      next if ( ! $_ );
      return_next( $_ );
  }

  return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 210 - Task 1 - PL/PgSQL Implementation

*I cheated here:* since I'm not sure I've understood very well the implementations, and I'm in a rush, I simply delegated to PL/Perl the implementation!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc210.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
   SELECT pwc210.task1_plperl( l );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 210 - Task 2 - PL/PgSQL Implementation

*I cheated here:* since I'm not sure I've understood very well the implementations, and I'm in a rush, I simply delegated to PL/Perl the implementation!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc210.task2_plpgsql( l int[] )
RETURNS SETOF int
AS $CODE$
   SELECT pwc210.task2_plperl( l );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
