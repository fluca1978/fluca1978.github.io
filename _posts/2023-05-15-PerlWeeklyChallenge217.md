---
layout: post
title:  "Perl Weekly Challenge 217: Did I misunderstand?"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 217: Did I misunderstand?

This post presents my solutions to the [Perl Weekly Challenge 217](https://perlweeklychallenge.org/blog/perl-weekly-challenge-217/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 217 - Task 1 - Raku](#task1)
- [PWC 217 - Task 2 - Raku](#task2)
- [PWC 217 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 217 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 217 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 217 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 217 - Task 1 - Raku Implementation

The first task was about to find the fourth element in a sorted matrix made of integers.
As far as I understand from the examples, the matrix is flattened into an array, sorted naturally and then the fourth element is extracted. This made me implement it as an application accepting a flat array.

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems } ) {
    @n.sort()[ 3 ].say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 217 - Task 2 - Raku Implementation

Given a list of integers, find out their composition that provides the biggest integer as a result.
<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my $max = 0;
    for @list.permutations {
	   $max = $_.join if ( $_.join.Int > $max );
    }

    $max.say;
}

```
<br/>
<br/>

The idea is to search thru all the permutations of the input array, trying to see if the array can be `join`ed into a `$max` value.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 217 - Task 1 - PL/Perl Implementation

If the function accepts the flattened matrix, its implementation is really simple:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc217.task1_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $array ) = @_;
   return ( sort( $array->@* ) )[ 3 ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 217 - Task 2 - PL/Perl Implementation

A *all permutations* based implementation, where I use the `List::Permutor` module and hence the `plperlu` untrusted language:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc217.task2_plperl( int[] )
RETURNS int
AS $CODE$
   use List::Permutor;
   my ( $max ) = 0;
   my $engine = List::Permutor->new( $_[0]->@* );
   while ( my @current = $engine->next ) {
   	 $max = join( '', @current ) if ( join( '', @current ) > $max );
   }

   return $max;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 217 - Task 1 - PL/PgSQL Implementation

A single query in SQL:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc217.task1_plpgsql( a int[] )
RETURNS int
AS $CODE$
   SELECT v
   FROM unnest( a ) v
   ORDER BY 1
   LIMIT 1
   OFFSET 3;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The idea is to `unnest` the incoming array, making it a table, order the result by the values, get a single result at displacement 3.


<a name="task2plpgsql"></a>
## PWC 217 - Task 2 - PL/PgSQL Implementation

Again, a single query implementation:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc217.task2_plpgsql( a int[] )
RETURNS int
AS $CODE$
   SELECT string_agg( v.vv::text, '' )::int
   FROM ( SELECT vv
          FROM unnest( a ) vv
	  ORDER BY ( vv % 10 ) DESC  ) v;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

I `unnest` the array making it a table, and order the result by the remainder of `10`. Then I aggregate the result into a single tring, that is then cast to an integer. *This is not a perfect solution, since it assumes you don't specify numbers greater than `10`*.
