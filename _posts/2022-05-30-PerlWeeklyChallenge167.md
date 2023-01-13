---
layout: post
title:  "Perl Weekly Challenge 167: too much math!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 167: too much math!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 167](https://perlweeklychallenge.org/blog/perl-weekly-challenge-167/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)


Last, the solutions in PostgreSQL PL/PgSQL:

<br/>
- [Task 1 in PostgreSQL Pl/PgSQL](#task1plpgsql)
- [Task 2 in PostgreSQL Pl/PgSQL](#task2plpgsql)

<a name="task1"></a>
## PWC 167 - Task 1

Compute all *cyclic primes* within the rage of three digits. A cyclic prime is a number that is prime and has its digits, permutated towards left, that provide also all prime numbers.



<br/>
<br/>
```raku
sub is-circular-prime( Int $prime )
{
    return False if ! $prime.is-prime;

    my @digits = $prime.comb;
    my $found = True;
    for 1 ..^ @digits.elems {
        return False if ! ( |@digits[ $_ .. * - 1], |@digits[ 0 .. $_ - 1 ] ).join.Int.is-prime;
    }

    return True;
}


sub MAIN() {
    (100 .. 999).grep( { is-circular-prime( $_ ) } ).join( "\n" ).say;
}

 ```
<br/>
<br/>

All the work is done clearly from the `is-circular-prime` function, that checks if the input number is prime, and then computes the permutations by moving digits towards left and re-inserting the extracted digits on the end.
As soon as a non-prime number is detected, the function stops.


<a name="task2"></a>
## PWC 167 - Task 2

An implementation of the Gamma Function, quite complex to me!

<br/>
<br/>
```raku
my @coeffs = <
    1.000000000000000174663
    5716.400188274341379136
  -14815.30426768413909044
   14291.49277657478554025
  -6348.160217641458813289
   1301.608286058321874105
  -108.1767053514369634679
   2.605696505611755827729
  -0.7423452510201416151527e-2
   0.5384136432509564062961e-7
  -0.4023533141268236372067e-8
    >;

sub gamma( $z )
{
    if ( $z < 0.5 ) {
        pi / sin( pi * $z ) / gamma( $z - 1 );
    }
    else {
        sqrt( 2 * pi )
        * ( $z + 9 - 0.5) ** ( $z  - 0.5)
        * exp( -1 * ( $z + 9 - 0.5))
        * [+] @coeffs Z* 1, |map 1 / ($z + * ) , 0 .. *;
    }

}

sub MAIN( Int $value ) {
    gamma( $value ).say;
}

```
<br/>
<br/>

I have to say that is not something I can re-implement easily, and I have to search online about possible implementations trying to understand how to implement it.

<a name="task1plperl"></a>
## PWC 167 - Task 1 in PostgreSQL PL/Perl

Pretty much the same implementation of Raku, done with anonymous subroutines:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc167.task1_plperl()
RETURNS SETOF int
AS $CODE$

# utility function to check if a number is prime
my $is_prime = sub {
   my ($num) = @_;
   for ( 2 .. $num - 1 ) {
     return 0 if ( $num % $_ == 0 );
   }

  return 1;
};

# get all the permutations
my $compute_permutations = sub {
  my ($num) = @_;
  my @perms;
  my @digits = split //, $num;

  for ( 0 .. $#digits  ) {
      push @perms, join( '',        @digits[ $_ .. $#digits ], @digits[ 0 .. $_ - 1 ] );
  }

  return @perms;
};

# check if the given number is a
# cyclic prime
my $is_cyclic_prime = sub {
  my ($value) = @_;

  return 0 unless( $is_prime->( $value ) );
  for ($compute_permutations->( $value ) ) {
      return 0 unless( $is_prime->( $_ ) );
  }

  return 1;
};

for ( 100 .. 999 ) {
    return_next( $_ ) if $is_cyclic_prime->( $_ );
}

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 167 - Task 2 in PostgreSQL PL/Perl

A translation of what the Raku version does:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc167.task2_plperl( numeric )
RETURNS numeric
AS $CODE$

my @coeffs = qw/
    1.000000000000000174663
    5716.400188274341379136
  -14815.30426768413909044
   14291.49277657478554025
  -6348.160217641458813289
   1301.608286058321874105
  -108.1767053514369634679
   2.605696505611755827729
  -0.7423452510201416151527e-2
   0.5384136432509564062961e-7
  -0.4023533141268236372067e-8
    /;


my $sum_coeffs = 0;
$sum_coeffs += $_ for ( @coeffs );

my $gamma = sub {
   my ($z) = @_;
   my $pi = 3.1415;

    if ( $z < 0.5 ) {
        $pi / sin( $pi * $z ) / gamma( $z - 1 );
    }
    else {
        sqrt( 2 * $pi )
        * ( $z + 9 - 0.5) ** ( $z  - 0.5)
        * exp( -1 * ( $z + 9 - 0.5))
        *  do {
            my ($sum, $i) = (shift(@coeffs), 0);
            $sum += $_ / ($z + $i++) for @coeffs;
            $sum;
        };
    }

};


return $gamma->( $_[0] );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task1plpgsql"></a>
## PWC 167 - Task 1 in PostgreSQL PL/PgSQL

*not implemented*

<br/>
<br/>

``` sql

```
<br/>
<br/>


<a name="task2plpgsql"></a>
## PWC 167 - Task 2 in PostgreSQL PL/PgSQL

*not implemented*

<br/>
<br/>

``` sql

```
<br/>
<br/>
