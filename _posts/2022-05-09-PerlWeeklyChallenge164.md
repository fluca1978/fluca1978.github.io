---
layout: post
title:  "Perl Weekly Challenge 164: quick and easy"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 164: quick and easy

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 164](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0164/){:target="_blank"}.

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
## PWC 164 - Task 1

This task was about finding prime numbers less than `1000` that are palindromes too.
I decided to implement it as a `lazy gather` array implementation:



<br/>
<br/>
```raku
sub MAIN( Int $limit = 1000 ) {
    my @primes-palindrome = lazy gather {
        for 0 ..^ $limit {
            take $_ if $_.is-prime && $_.Str.flip == $_.Str && $_.Str.comb.chars > 1;
        }
    };

    .say for @primes-palindrome;
}

 ```
<br/>
<br/>

It simply checks for a number to be prime, if its stringified representtion is equal to its flipped (i.e. reversed) repreentation, and to have at least two chars.


<a name="task2"></a>
## PWC 164 - Task 2

find the first eight happy numbers, where a number is happy if it can be reduced to `1` by summing the squares of its digits:

<br/>
<br/>
```raku
sub MAIN( Int $limit = 8 ) {
    my @happy-numbers = lazy gather {
        for 10 .. Inf {
            my $sum = $_.comb.map( * ** 2 ).sum;
            while ( $sum.comb.elems > 1 ) {
                $sum = $sum.comb.map( * ** 2 ).sum;
            }

            take $_ if $sum == 1;
        }
    };

    @happy-numbers[ 0 .. $limit ].join( ', ' ).say;
}

```
<br/>
<br/>

Again, I decided to implement with a `lazy gather` approach and an infinite loop.
Within the loop, I reduce the number to its `$sum`, that is the sum of the squares of its digits. I reiterate the approach while the `$sum` has more than one digit, and when it finally has a single digit, I return the number only if it is equal to `1`.

## PWC 164 - Task 1 in PostgreSQL PL/Perl

A simple implementation using two anonymous subroutines to check if a number is prime and if it is palindrome:


<br/>
<br/>

``` perl
CREATE OR REPLACE FUNCTION
pwc164.task1_plperl( int )
RETURNS SETOF int
AS $CODE$
  my ( $limit ) = @_;

  my $is_prime = sub {
     my ( $number ) = @_;
     for my $i ( 2 .. $number - 1 ) {
         return 0 if $number % $i == 0;
     }

     return 1;
  };

  my $is_palindrome = sub {
     return $_[ 0 ] eq reverse $_[ 0 ];
  };

  for  ( 10 .. $limit ) {
      return_next( $_ ) if $is_prime->( $_ ) && $is_palindrome->( $_ );
  }

  return undef;
$CODE$
LANGUAGE plperl;
```
<br/>
<br/>

The function returns a new number when found using `return_next`.

<a name="task2plperl"></a>
## PWC 164 - Task 2 in PostgreSQL Pl/Perl

I created an anonymous subroutine that, given a number, reduces it to a single digit and provides a boolean value if the number is happy. Than it is a matter of looping before returning a new number:


<br/>
<br/>
``` sql
CREATE OR REPLACE FUNCTION
pwc164.task2_plperl( int )
RETURNS SETOF int
AS $CODE$
  my ( $limit ) = @_;

  my $is_happy = sub {
     my ( $num ) = @_;
     my $sum = $num;
     while ( $num > 10 ) {
             $sum = 0;
             $sum += $_ for map { $_ ** 2 }  split( //, $num );
             $num = $sum;
             $sum = 0;
    }

    return $num == 1;
  };

  for ( 10 .. 99999 ) {
      $limit-- and return_next( $_ )  if $is_happy->( $_ );
      last if $limit == 0;
  }

  return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

It is interesting to note that I do loop until `$limit` reaches zero, and `$limit` is decreased before `return_next` is executed thanks to the low priority `and`.


## PWC 164 - Task 1 in PostgreSQL Pl/PgSQL

Same implementation as PL/Perl, but this time the function to understand if a number is prime is a separate one:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc164.is_prime( n int )
RETURNS boolean
AS $CODE$
DECLARE
BEGIN
   FOR i IN 2 .. n - 1 LOOP
     IF n % i = 0 THEN
       RETURN false;
     END IF;
   END LOOP;

   RETURN true;
END
$CODE$
LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION
pwc164.task1_plpgsql( l int DEFAULT 1000 )
RETURNS SETOF INT
AS $CODE$
DECLARE
BEGIN

     FOR n IN 10 .. l LOOP
         IF pwc164.is_prime( n ) AND n = reverse( n::text )::int THEN
            RETURN NEXT n;
            l := l - 1;
         END IF;

         EXIT WHEN l = 0;
     END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>

Luckily, there is a built-in `reverse` function that can be used to check if the number, converted to a string, is equal to the starting number.

<a name="task2plpgsql"></a>
## PWC 164 - Task 2 in PostgreSQL Pl/PgSQL

Same implementation as PL/Perl, but with a standalone function to check if the number is happy:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc164.is_happy( n int )
RETURNS bool
AS $CODE$
DECLARE
        s  int;
        nn int;
BEGIN
        WHILE n > 10 LOOP
              s := 0;
              FOREACH nn IN ARRAY regexp_split_to_array( n::text, '' )::int[] LOOP
                s := s + nn * nn;
              END LOOP;

              n := s;

        END LOOP;

        IF n = 1 THEN
           RETURN true;
        ELSE
           RETURN false;
        END IF;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc164.task2_plpgsql( l int DEFAULT 8 )
RETURNS SETOF INT
AS $CODE$
DECLARE
BEGIN
        FOR n IN 10 .. 99999 LOOP
            IF pwc164.is_happy( n ) THEN
               RETURN NEXT n;
               l := l - 1;
            END IF;

            EXIT WHEN l = 0;
        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I use a `regexp_split_to_array()` call to get all the digits of the number to check, than I iterate on every digit and do the sum `s`. I repeat the same workflow until the number has a single digit, in such case I can test for its happyness.
<br/>
The other function simply represents the entry point and does the looping, then returning every happy number found.
