---
layout: post
title:  "Perl Weekly Challenge 362: Lingua to the rescue!"
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

# Perl Weekly Challenge 362: Lingua to the rescue!

This post presents my solutions to the [Perl Weekly Challenge 362](https://perlweeklychallenge.org/blog/perl-weekly-challenge-362/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 362 - Task 1 - Raku](#task1)
- [PWC 362 - Task 2 - Raku](#task2)
- [PWC 362 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 362 - Task 2 in PostgreSQL PL/Perl](#task2plperl)


# Raku Implementations

<a name="task1"></a>
## PWC 362 - Task 1 - Raku Implementation

Given a string made of letters, produce another string repeating any letter for the times of its index.

<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / ^ <[a..z]>+ $ / } ) {
    my @chars  = $string.comb;
    my $result = "";

    for 0 ..^ @chars.elems {
		$result ~= @chars[ $_ ] x ( $_ + 1 );
    }

    $result.say;
}

```
<br/>
<br/>


I use the `x` char repeator to the aim.

<a name="task2"></a>
## PWC 362 - Task 2 - Raku Implementation

Given an array of numbers, print them out in order of their phonetic pronunciation.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    my %symbols =
		  0 => 'zero',
		  1 => 'one',
		  2 => 'two',
		  3 => 'three',
		  4 => 'four',
		  5 => 'five',
		  6 => 'six',
		  7 => 'seven',
		  8 => 'eight',
		  9 => 'nine',
		  10 => 'ten',
		  11 => 'eleven',
		  12 => 'twelve',
		  13 => 'thirteen',
		  14 => 'fourteen',
		  15 => 'fifteen',
		  16 => 'sixteen',
		  17 => 'seventeen',
		  18 => 'eighteen',
		  19 => 'nineteen',
		  20 => 'twenty',
    ;

    my @phonetic;

    for @nums {
		my $current = 'plus ';

		if ( $_.Int < 0 ) {
		    $current = 'minus ';
		}

		my $value = $_.Int.abs;
		my $x;
		if ( $value >= 1000 ) {
		    ( $x, $value ) = $value / 1000, $value % 1000;
		    $current ~= ' ' ~ %symbols{ $x.Int } ~  ' thousands'
		}


		if ( $value >= 100 ) {
		    ( $x, $value ) = $value / 100, $value % 100;
		    $current ~= ' ' ~ %symbols{ $x.Int } ~  ' hundreds'
		}

		if ( $value > %symbols.keys.max ) {
		    ( $x, $value ) = $value / 10, $value % 10;
		    $current ~= ' ' ~ %symbols{ $x.Int } ~  'y'
		}

		if ( $value <= %symbols.keys.max ) {
		    $current ~= ' ' ~ %symbols{ $value.Int };
		}


		@phonetic.push: $current;
    }

    @phonetic.sort.join( "\n" ).say;

}

```
<br/>
<br/>

This is ugly and wrong!
A `Lingua` alike module should be used in here, while I simply try to decode the number into a *Not-so-English* form, sorting them the results and printing them out.

Moreove, after pushing the solution, I found that I had to print the number values, not their pronunciation. Well, this is quite simple: put the pronunciation as the key of an hash and the number as the value, and traverse the hash by sorting the keys.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 362 - Task 1 - PL/Perl Implementation

Same implementation as Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc362.task1_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;

   my $i = 0;
   my $result = "";

   for my $c ( split //, $string ) {
       $result .= $c x ( ++$i );
   }

   return $result;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 362 - Task 2 - PL/Perl Implementation

Here I use the `Lingua::EN::Numbers` that is the only correct way of solving this task.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc362.task2_plperl( int[] )
RETURNS SETOF text
AS $CODE$

   use Lingua::EN::Numbers qw(num2en num2en_ordinal);

   my ( $nums ) = @_;

   my @phonetic;

   for ( $nums->@* ) {
       push @phonetic, num2en( $_ );
   }

   @phonetic = sort @phonetic;
   return \ @phonetic;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


Again, shame on me, I had to print out the values, not their english form, so I should have used an hash keyed by the english form and then iterate on it with sorted keys.
