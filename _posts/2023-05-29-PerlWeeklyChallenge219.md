---
layout: post
title:  "Perl Weekly Challenge 219: building a tree using a stack"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 219: building a tree using a stack

This post presents my solutions to the [Perl Weekly Challenge 219](https://perlweeklychallenge.org/blog/perl-weekly-challenge-219/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 219 - Task 1 - Raku](#task1)
- [PWC 219 - Task 2 - Raku](#task2)
- [PWC 219 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 219 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 219 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 219 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 219 - Task 1 - Raku Implementation

The first task was effectively simple: given a list of integers, sort the squares.

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems } ) {
    @n.map( { $_ ** 2 } ).sort.join( ', ' ).say;
}

```
<br/>
<br/>

It can be solved with a single line: build an array of squares (using `map`), `sort` them and print.


<a name="task2"></a>
## PWC 219 - Task 2 - Raku Implementation

This has been a lot more difficult: given an array of costs for 1, 7 and 30 days, and a list of days (of year), compute the minimum cost of the vacancy.

<br/>
<br/>
```raku
sub MAIN() {

    my @days  = 1, 5, 6, 7, 9, 15;

    # sort the days
    @days .= sort;

    my %costs;
    %costs<1> = 2;
    %costs<7> = 7;
    %costs<30> = 25;

    my @evaluated;
    @evaluated.push: { cost => 0, days => @days };
    my $current-cost = @days.elems * %costs<1>;

    while ( @evaluated.elems > 0 ) {
		my %entry = @evaluated.shift;

		if ( %entry<days>.elems == 0 ) {
		    $current-cost = %entry<cost> if ( %entry<cost> < $current-cost );
		}
		else {
		     next if ( %entry<cost> >= $current-cost );

		    my $begin-date = %entry<days>[ 0 ];
		    for %costs.keys {
					my $end-date = $begin-date + $_ - 1;
					my @uncovered-days = %entry<days>.grep( * > $end-date );
					my $cost = %entry<cost> + %costs{ $_ };
					@evaluated.push: { cost => $cost, days => @uncovered-days };
		    }
		}
    }

    say $current-cost;
}

```
<br/>
<br/>

First of all, I define an hash named `%costs` that is keyed by the number of days and has the cost as a value.
Assuming the daily cost is always the lowest one, I compute the cost of single days and push them as an hash into the `@evaluated` array.
Then I loop in seeking for a solution: at every iteration I extract an hash element from the array, `%entry`. Such element has the cost and the list of remaining days: if the latter is empty there is nothing more to compute and so I simply evaluate the final cost to see if I've found a min.
Otherwise, there are other days to evaluate, so I consider the cost of this `%entry` and skip if it is already higher than what I have. Otherwise, I compute the final date of all the three periods of time, and the cost, and add the result to the `@evaluated` array, so that I'm building a kind of tree of possibile solutions.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 219 - Task 1 - PL/Perl Implementation

The same solution as the Raku implementation, with the only difference that I do an iterative `return_next` to handle big arrays:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc219.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $n ) = @_;
   for my $value ( sort { $a <=> $b }  map { $_ * $_ } $n->@* ) {
       return_next( $value );
   }
return;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 219 - Task 2 - PL/Perl Implementation

Same solution as Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc219.task2_plperl( int[], int[] )
RETURNS int
AS $CODE$
   my ( $c, $days ) = @_;
   my $costs = {};
   $costs->{ 1 }  = $c->@[ 0 ];
   $costs->{ 7 }  = $c->@[ 1 ];
   $costs->{ 30 } = $c->@[ 2 ];

   my @evaluated;
   my $current_cost = ( scalar $days->@* ) * $costs->{ 1 };
   push @evaluated, { cost => 0, days => $days };

   while ( ( scalar @evaluated ) > 0 ) {
   	 my $entry = shift @evaluated;

	 if ( $entry->{ days }->@* == 0 ) {
	    $current_cost = $entry->{ cost } if ( $entry->{ cost } < $current_cost );
	 }
	 else {
	      next if ( $entry->{ cost } >= $current_cost );

	      my $begin_date = $entry->{ days }->[ 0 ];
	      for my $duration ( keys $costs->%* ) {
	      	  my $end_date = $begin_date + $duration - 1;
		  my $cost = $entry->{ cost } + $costs->{ $duration };
		  my @remaining_days =  grep { $_ > $end_date }  $entry->{ days }->@*;
		  push @evaluated, { cost => $cost, days => \@remaining_days };
	      }
	 }
   }

   return $current_cost;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 219 - Task 1 - PL/PgSQL Implementation

A single query to do the trick!


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc219.task1_plpgsql( n int[] )
RETURNS SETOF int
AS $CODE$
   SELECT v * v
   FROM unnest( n ) v
   ORDER BY 1
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 219 - Task 2 - PL/PgSQL Implementation

Here *I cheat: I call the PL/Perl implementation* because I have no idea about how to translate this to PL/PgSQL in a decent way!


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc219.task2_plpgsql( c int[], days int[] )
RETURNS int
AS $CODE$
   SELECT pwc219.task2_plperl( c, days );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
