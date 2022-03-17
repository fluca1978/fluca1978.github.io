---
layout: post
title:  "Perl Weekly Challenge 156: Pernicious weirdness"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 156: Pernicious weirdness

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 156](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0156/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)



## COVID-19

Yes, I've *Covid-19* and I've done this PWC late because I was sick.
I'm not scared, at the moment I'm almost fine.


<a name="task1"></a>
## PWC 156 - Task 1

Produce the first ten *pernicious* numbers, that are numbers that have the sum of their binary representation that is prime number. It is quite easy to implement this in Raku:




<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 0 } = 10 ) {
    my @pernicious = lazy gather {
	for 1 .. Inf {
	    take $_ if $_.base( 2 ).comb.sum.is-prime;
	}
    };

    @pernicious[ 0 .. $limit - 1 ].join( ', ' ).say;
}
 ```
<br/>
<br/>
 I use a `lazy gather` block to generate the array of required numbers. Every number is translated into its binary form by means of `base( 2 )`, separated into its digits by means of `comb` and the sum of all the binary digits is made up by `sum`. If the result is a prime number, i.e., `is-prime`, the result is `take`n into the array.
 <br/>
 In the end I print the slice of the array.


<a name="task2"></a>
## PWC 156 - Task 2

Weird numbers this time: numbers that have a sum of divisors that is greater than the number itself and that cannot be obtained by any sum of any divisors.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {
    my @divisors = ( 1 .. $n - 1 ).grep( $n %% * );
    '0'.say and exit if @divisors.sum <= $n
		     || @divisors.combinations.map( *.sum ).grep( * == $n ).elems > 0;
    '1'.say;
}

```
<br/>
<br/>

As first step, I compute the `@divisors` by means of `grep`ping all the values that are divisors of the given number.
Then, I inspect the `sum` of the `@divisors` to see if it is greater than the number itself, and then use all `combinations` of the divisors to see if any of the `sum` provides the number itself, in such case the number is not weird.


<a name="task1pg"></a>
## PWC 156 - Task 1 in PostgreSQL


An implementation with a couple of PL/Perl functions and a single query.

Pure Pl/Perl implementation, with a single function (that has nested anonymous `sub`s).


<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc156;

CREATE OR REPLACE FUNCTION
pwc156.is_prime( int )
RETURNS bool
AS $CODE$
   return 0 if $_[0] <= 1;
   for my $i ( 2 .. $_[0] - 1 ) {
       return 0 if $_[0] % $i == 0;
   }

   return 1;
$CODE$
LANGUAGE plperl;


CREATE OR REPLACE FUNCTION
pwc156.sumbits( int[] )
RETURNS int
AS $CODE$
   my $sum = 0;
   for my $bit ( @{$_[0]} ) {
       $sum += $bit;
   }

   return $sum;
$CODE$
LANGUAGE plperl;



SELECT n
FROM generate_series( 1, 1000 ) n
WHERE pwc156.is_prime( pwc156.sumbits( regexp_split_to_array( n::bit(10)::text, '' )::int[] ) )
LIMIT 10;
```
<br/>
<br/>

The first function is the usual `is_prime`, built to return the right values for the given input.
<br/>
The `sumbits` function accepts an array of digits and returns its sum. It is interesting to note that the PostgreSQL `ARRAY` is not the same as a Perl list, and in fact is managed by PL/Perl by a specific object that can be used as an array reference, hence `@{ $_[0] }`.
<br/>
The query does all the work:
- `generate_series` provides the first thousand integers;
- the cast to `::bit(10)` converts the integer into its binary representation;
- `regexp_slit_to_array` converts the text binary representation into an array of integers;
- `sumbits` computes the sum of the bits;
- `is_prime` sees if the sum is a prime number.



<a name="task2pg"></a>
## PWC 156 - Task 2 in PostgreSQL

Here I decided to use a library, hence the need for `plperlu` to load an external module. The module is `Algorithm::Knapsack` that can provide a *brute force* implementation of the knapsack problem. The idea is that you can initialize the knapsack with a capacity and a set of weights, and the system computes all possible solutions that fill the bag at its maximum. The idea therefore is to see if any of these solutions fill the whole capacity, in our case the input number.


<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc156;

CREATE OR REPLACE FUNCTION
pwc156.weird( int )
RETURNS bool
AS $CODE$

use Algorithm::Knapsack;
my @divisors;

for my $i ( 2 .. $_[0] - 1 ) {
   push @divisors, $i if $_[0] % $i == 0;
}

my $sum = 0;
for my $i ( @divisors ) {
   $sum += $i;
}


return 0 if $sum <= $_[0];

my $knapsack = Algorithm::Knapsack->new(
    capacity => $_[0],
    weights  => \@divisors,
);
 
$knapsack->compute();
 
foreach my $solution ($knapsack->solutions()) {
    my @founds = @divisors[ $solution->@* ];
    my $sum = 0;
    $sum += $_ for ( @founds );
    return 0 if $sum == $_[0];
 
}


return 1;
$CODE$
LANGUAGE plperlu;




SELECT n
FROM generate_series( 1, 100 ) n
WHERE pwc156.weird( n );
```
<br/>
<br/>


The `weird` function does everything:
- computes the array of the `@divisors`;
- initializes the `$knapsack` and computes the solutions;
- then inspect every solution to see if the sum of the divisors provides back the input number.

<br/>
The query does everything, testing the implementation.