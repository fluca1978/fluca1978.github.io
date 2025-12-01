---
layout: post
title:  "Perl Weekly Challenge 350: only Perl!"
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

# Perl Weekly Challenge 350: only Perl!

This post presents my solutions to the [Perl Weekly Challenge 350](https://perlweeklychallenge.org/blog/perl-weekly-challenge-350/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 350 - Task 1 - Raku](#task1)
- [PWC 350 - Task 2 - Raku](#task2)
- [PWC 350 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 350 - Task 2 in PostgreSQL PL/Perl](#task2plperl)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 350 - Task 1 - Raku Implementation

Given a string, split it into chunks of `3` chars, and see if these chunks are *good strings*, meaning they don't have a repeated char in it.


<br/>
<br/>
```raku
sub MAIN( Str $string, Int $size = 3 ) {

    my @chars = $string.comb;
    my @good-substrings;

    for 0 .. @chars.elems - $size -> $index {
		my $current = Bag.new: @chars[ $index ..^ $index + $size ];
		next if $current.values.grep( * > 1 );
		@good-substrings.push: $current.keys.join;
    }

    @good-substrings.elems.say;
}

```
<br/>
<br/>


I first extract all the `@chars` into an array, and then I iterate over every single array position up to `elems - $size` to stop three position before the end of the array. I place all the chars into a `Bag`, so that I automatically count all the occurrencies of every char.
If the bag has a value that is more than one, a char is repeated, so simply continue to the next chunk. If not, the string is good so place into the `@good-substrings` array and then count the elements.


<a name="task2"></a>
## PWC 350 - Task 2 - Raku Implementation

The second task is to find good pairs of numbers, assuming the two pairs are made by the same digits in a different order and the first number is a divisor of the second.


<br/>
<br/>
```raku
sub MAIN( Int $from, Int $to, Int $count
	where { $to > $from && $count >= 1 } ) {

    my %found-pairs;
    for $from .. $to -> $current {
		my @pairs = $current.comb.permutations
			     .map( *.join.Int )
			     .grep( * > $current )
			     .grep( * %% $current );


		next if ! @pairs;
		%found-pairs{ $current } = @pairs;

    }

    %found-pairs.values.elems.say;
}

```
<br/>
<br/>

I iterate over all the numbers in the range `$from` to `$to`, and then compute all the `permutations` and `map` such arrays into a single integer value, then keep only the numbers greater than the `$current` and that are divisible by the `$current` number in the list.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 350 - Task 1 - PL/Perl Implementation

Similar to Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc350.task1_plperl( text, int )
RETURNS int
AS $CODE$

   my ( $string, $size ) = @_;
   $size //= 3;

   my @chars = split //, $string;
   my @found;

   for my $index ( 0 .. $#chars - $size + 1 ) {
       my $bag = {};
       for ( 0 .. $size - 1 ) {
       	   $bag->{ $chars[ $_ + $index ] }++;
       }

       my $ok = 1;
       for ( keys $bag->%* ) {
       	   $ok = 0 if ( $bag->{ $_ } > 1 );
       }

       next if ! $ok;
       push @found, join '', keys $bag->%*;
   }

   return scalar @found;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The idea is the same as in Raku, using an hash as a bag.


<a name="task2plperl"></a>
## PWC 350 - Task 2 - PL/Perl Implementation


Here I need `List::Permutor` to compute all the permutations of the digits of a given number.
Therefore, I use the `plplerlu` language, hence the need of permissions from an administrator user.
The implemnetation is the same as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc350.task2_plperl( int, int, int)
RETURNS int
AS $CODE$

   my ( $from, $to, $count ) = @_;
   my $found = {};

   use List::Permutor;


   for my $current ( $from .. $to ) {
       my $digits = List::Permutor->new( split //, $current );
       while ( my @permutations = $digits->next ) {
       	     my $comparing = join( '', @permutations );
	     if ( $current < $comparing && $comparing % $current == 0 ) {
	     	push $found->{ $current }->@*, $comparing;
	     }
       }
   }

   return scalar values $found->%*;



$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

