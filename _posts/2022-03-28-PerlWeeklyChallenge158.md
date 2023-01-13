---
layout: post
title:  "Perl Weekly Challenge 158: to be prime or not to be prime"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 158: to be prime or not to be prime

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 158](https://perlweeklychallenge.org/blog/perl-weekly-challenge-158/){:target="_blank"}.

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


## COVID-19

I'm fine, and I'm working from home for another couple of days at least.



<a name="task1"></a>
## PWC 158 - Task 1

Really simple, just one line of code. The task required to find out all prime numbers those digits, when summed, provide another prime number. It was a matter of `grep`ping the data:




<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 0 } = 100 ) {
    ( 1 .. $limit ).grep( *.is-prime ).grep( *.comb.sum.is-prime ).say;
}
 ```
<br/>
<br/>



<a name="task2"></a>
## PWC 158 - Task 2

A *Cuban prime* is a number that when elevated at its cube and subtracted by the very former one (also cubed) gives a prime. Assuming this can be some time expensive, let's do with a `lazy` and `gather` block:

<br/>
<br/>
```raku
sub MAIN( Int $limit  where { $limit > 0 } = 1000 ) {
    my @cuban-primes = lazy gather {
        for ( 1 .. Inf ) {
            my $cuban = ( $_ + 1 ) ** 3 - $_ ** 3;
            last if $cuban > $limit;
            take $cuban if $cuban.is-prime;
        }
    };

    @cuban-primes.join( ', ' ).say;
}

```
<br/>
<br/>



<a name="task1plperl"></a>
## PWC 158 - Task 1 in PostgreSQL PL/Perl


A pure Perl implementation using anonymous code blocks:


<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc158;

CREATE OR REPLACE FUNCTION
pwc158.add_primes( int )
RETURNS SETOF int
AS $CODE$

# function to test if a number is prime
my $is_prime = sub {
   my ($value) = @_;
   for my $i ( 2 .. $value - 1 ) {
     return 0 if ( $value % $i == 0 );
   }

  return 1;
};

# disassemble a number into its digits and sum them
my $sum_digits = sub {
   my ($value) = @_;
   my @digits = split //, $value;
   my $sum = 0;
   $sum += $_ for ( @digits );
   return $sum;
};


for my $i ( 1 .. $_[0] ) {
    return_next( $i ) if ( $is_prime->( $i )
                        && $is_prime->( $sum_digits->( $i ) ) );
}

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The idea is quite simple: in the last loop there is the test for a number to be prime as a standalone and as the sum of its digits. In the case the test is fine, the number is appended to the result set.



<a name="task2plperl"></a>
## PWC 158 - Task 2 in PostgreSQL Pl/Perl

Similar to the above implementation, I decided to use a single function with anonymous code blocks to test for a number to be prime and sum all the digits:

<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc158;

CREATE OR REPLACE FUNCTION
pwc158.cuban_primes( int )
RETURNS SETOF int
AS $CODE$

# function to test if a number is prime
my $is_prime = sub {
   my ($value) = @_;
   for my $i ( 2 .. $value - 1 ) {
     return 0 if ( $value % $i == 0 );
   }

  return 1;
};

for ( 1 .. 999999 ) {
    my $cuban = ( $_ + 1 ) ** 3 - $_ ** 3;
    last if $cuban >= $_[ 0 ];
    return_next( $cuban ) if $is_prime->( $cuban );
}

return undef;
$CODE$
LANGUAGE plperl;


```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 158 - Task 1 in PostgreSQL Pl/PgSQL

I created a couple of utility functions to test for a number to be prime and to sum all its digits.
Then a simple query can suffice to the aim:

<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc158;

CREATE OR REPLACE FUNCTION
pwc158.is_prime( v bigint )
RETURNS bool
AS $CODE$
DECLARE
i int;
BEGIN
FOR i IN  2 .. v - 1  LOOP
IF ( v % i = 0 ) THEN
RETURN false;
END IF;
END LOOP;

RETURN true;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc158.sum_digits( n bigint )
RETURNS int
AS $CODE$
   WITH s(i) AS ( SELECT regexp_split_to_table( n::text, '' ) )
   SELECT sum( i::bigint ) FROM s;
$CODE$
LANGUAGE SQL;


SELECT n
FROM generate_series( 1, 100 ) n
WHERE pwc158.is_prime( n )
AND pwc158.is_prime( pwc158.sum_digits( n ) );

```
<br/>
<br/>

In order to sum the digits of a number I decided to use a CTE that converts the number into a table representation of its digits, and then use the built-in SQL `sum` function to achieve the trick.



<a name="task2plpgsql"></a>
## PWC 158 - Task 2 in PostgreSQL Pl/PgSQL

Again, a couple of utility functions to see if a number is prime and compute the difference between the cuban sequence.
At last, a query can solve the problem.

<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc158;


CREATE OR REPLACE FUNCTION
pwc158.is_prime( v bigint )
RETURNS bool
AS $CODE$
DECLARE
        i int;
BEGIN
        FOR i IN  2 .. v - 1  LOOP
            IF ( v % i = 0 ) THEN
               RETURN false;
            END IF;
        END LOOP;

        RETURN true;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc158.cuban( v bigint )
RETURNS bigint
AS $CODE$
   SELECT pow( v + 1, 3 ) - pow( v, 3 );
$CODE$
LANGUAGE sql;



SELECT pwc158.cuban( v )
FROM generate_series( 1, 100 ) v
WHERE pwc158.is_prime( pwc158.cuban( v ) )
;

```
<br/>
<br/>

Here I use the SQL `pow` function to compute the difference between the cubes.
There is the execution of `cuban` twice, and this could be optimized with a CTE.
