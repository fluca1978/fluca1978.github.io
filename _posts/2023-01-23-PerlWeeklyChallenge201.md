---
layout: post
title:  "Perl Weekly Challenge 201: not satisfied!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 201: not satisfied!

This post presents my solutions to the [Perl Weekly Challenge 201](https://perlweeklychallenge.org/blog/perl-weekly-challenge-201/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 201 - Task 1 - Raku](#task1)
- [PWC 201 - Task 2 - Raku](#task2)
- [PWC 201 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 201 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 201 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 201 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 201 - Task 1 - Raku Implementation

The first task was about searching for missing numbers in a given list of scattered (i.e., unordered) numbers. This is a matter of `grep`ping every number in a range:

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ) == @n.elems } ) {
    my @missing-numbers;
    for 0 ^.. @n.elems {
    	@missing-numbers.push: $_ if ( ! @n.grep( $_ ) );
    }

    @missing-numbers.join( ', ' ).say;

}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 201 - Task 2 - Raku Implementation

The second task is much more complex than the first one, and it involves *computing the integer partitions* for a given input number. It turned out that this task is too much difficult for me to solve, and that's why I'm not satisfied in using an external library to achieve the result. However, thanks to `Math::Combinatorics`, I can use the `partitions` method that gives me the result:

<br/>
<br/>
```raku
use Math::Combinatorics <partitions>;

sub MAIN( Int $n where { $n > 0 } ) {
    say partitions( $n ).elems;
}
```
<br/>
<br/>


# PL/Perl Implementations

A very similar implementation to the Raku one, so I do `grep` for a given number in the input list:

<a name="task1plperl"></a>
## PWC 201 - Task 1 - PL/Perl Implementation

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc201.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $n ) = @_;

   for my $needle ( 1 .. $n->@* ) {
       return_next( $needle ) if ( ! grep( { $_ == $needle } $n->@* ) );
   }

return;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 201 - Task 2 - PL/Perl Implementation

Again, I had to involve an external library, this time `Integher::Partitions`.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc201.task2_plperl( int)
RETURNS int
AS $CODE$

  use Integer::Partition;
  my $partitions = Integer::Partition->new( $_[0] );
  my $count = 0;
  $count++  while( $partitions->next );
  return $count;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

This means that the code has to be written as `plperlu`, i.e., it is *untrusted*. Moreover, the `Integer::Partiton` library does provide only an iterative interface, so I need to manually count all the partitions before returning the result.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 201 - Task 1 - PL/PgSQL Implementation

The first task can be solved with a simple SQL query:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc201.task1_plpgsql( n int[] )
RETURNS SETOF int
AS $CODE$
   SELECT v
   FROM generate_series( 1, array_length( n, 1 ) ) v
   WHERE v NOT IN
   ( SELECT unnest( n ) );
$CODE$
LANGUAGE sql;
```
<br/>
<br/>

The function accepts an array and transforms it into a table by means of the `unnest` function. The result is then joined with the result of `generate_series`, a function that produces a sequences of integers. I keep only the tuples that do not match, meaning I'm keeping all the numbers that are not in the original array.


<a name="task2plpgsql"></a>
## PWC 201 - Task 2 - PL/PgSQL Implementation

Again, I'm not satisfied of this solution! Since it is not simple to achieve this in SQL, I decided to simply call the PL/Perl implementation, that in turns calls an external module function, and this means there is nothing out of my bag here!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc201.task2_plpgsql( n int )
RETURNS int
AS $CODE$
   SELECT pwc201.task2_plperl( n );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
