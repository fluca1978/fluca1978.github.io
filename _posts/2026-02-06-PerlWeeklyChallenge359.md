---
layout: post
title:  "Perl Weekly Challenge 359: quick and dirty!"
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

# Perl Weekly Challenge 359: quick and dirty!

This post presents my solutions to the [Perl Weekly Challenge 359](https://perlweeklychallenge.org/blog/perl-weekly-challenge-359/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 359 - Task 1 - Raku](#task1)
- [PWC 359 - Task 2 - Raku](#task2)
- [PWC 359 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 359 - Task 2 in PostgreSQL PL/Perl](#task2plperl)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 359 - Task 1 - Raku Implementation

Given a number, compute the number of time the digits have to be summed to get a single digit number and what this number is.



<br/>
<br/>
```raku
sub MAIN( Int $number ) {
    my $persistence = 0;
    my $root = $number;

    while ( $root >= 10 ) {
		$root = [+] $root.Str.comb.map( *.Int );
		$persistence++;
    }

    "Persistence : $persistence \nRoot : $root".say;

}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 359 - Task 2 - Raku Implementation

Given a string, remove all adiacent duplicated characters.


<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / ^ <[a..zA..Z]>+ $ / } ) {

    my $reduced = $string;
    $reduced ~~ s:g/ (.) $0 //;
    $reduced.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 359 - Task 1 - PL/Perl Implementation

Straightfoward with the usage `split`:


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc359.task1_plperl( int )
RETURNS SETOF INT
AS $CODE$

   my ( $number ) = @_;

   my $root        = $number;
   my $persistence = 0;


   while ( $root > 9 ) {
   	 my $sum = 0;
	 for ( split //, $root ) {
	     $sum += $_;
	 }

	 $root = $sum;
	 $persistence++;
   }

   return [ $persistence, $root ];


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 359 - Task 2 - PL/Perl Implementation

Use the same regular expression approach as in Raku:


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc359.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;

   $string =~ s/ (.) \1 //gx;
   return $string;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

