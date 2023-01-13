---
layout: post
title:  "Perl Weekly Challenge 168: prime numbers in many ways!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 168: prime  umbers in many ways!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 168](https://perlweeklychallenge.org/blog/perl-weekly-challenge-168/){:target="_blank"}.

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
## PWC 168 - Task 1

Compute first 13 Perrin prime numbers, that are numbers in the Perrin sequence that are also primes.
At glance, I tried to use the Raku sequence `...` operator with something like the following:



<br/>
<br/>
```raku
my @perrin = 3, 0, 2, -> $left, $right { $left + $right } ... *;
my @perrin-primes = @perrin.grep( *.is-prime );
@perrin-primes.unique.head( 3 ).join( "\n" ).say;
 ```
<br/>
<br/>

However, this requires too much time on my machine, so I produced something more "classical" as the following:

<br/>
<br/>

``` raku
sub MAIN( Int $limit where { $limit > 1 } = 13 ) {
    my @perrin = 3, 0, 2;
    my @perrin-primes;

    while ( @perrin-primes.elems < $limit ) {
        @perrin.push: @perrin[ * - 2 ] + @perrin[ * - 3 ];
        @perrin-primes.push: @perrin[ * - 1 ] if @perrin[ * - 1 ].is-prime && ! @perrin-primes.grep( @perrin[ * - 1 ] );
    }

    @perrin-primes.sort.join( "\n" ).say;
}
```
<br/>
<br/>

The idea is to populate the `@perrin` array with the sequence, and then put a new prime number inot the `@perrin-primes` only if not already inserted (Perrin numbers can repeat).


<a name="task2"></a>
## PWC 168 - Task 2

Compute the *home prime* value of a given input number. A Home Prime number is the prime number obtained out of the sequence of using factors of the input numbers as a compound digit.

<br/>
<br/>
```raku
sub prime-factors( Int $n where { $n > 0 } )
{
    my $number = $n;
    my @factors;
    return $n if $n.is-prime;

    for 2 ..^ $n {
        next if ! $_.is-prime;
        next if $number !%% $_;
        next if $_ > $number;

        while ( $number %% $_ ) {
            @factors.push: $_;
            $number /= $_;
        }

    }

    return @factors;
}

sub HP( Int $n where { $n > 0 } )
{
    my $number = prime-factors( $n ).join.Int;
    return $number if $number.is-prime;
    return HP( $number );
}

sub MAIN( Int $n where { $n > 1 } = 10 ) {
    say HP( $n );
}


```
<br/>
<br/>

As you can see, the `MAIN` is really simple and it calls only the `HP` function.
The `HP` function in turn invokes the `prime-factors` on its input to get the array of prime factors. For example, `prime-factors(10)` gives the array with `2,5`. The `HP` composes the prime factors into a single digit (e.g., `2, 5` becomes `25`) and test if the number is prime, otherwise recursively calls itself.

<a name="task1plperl"></a>
## PWC 168 - Task 1 in PostgreSQL PL/Perl

Straightforward implementation using an anonymous function to see if a number is prime:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc168.task1_plperl( int )
RETURNS SETOF bigint
AS $CODE$
  my ( $limit ) = @_;
  $limit //= 13;
  my @perrin = (3, 0, 2);
  my $seen = {};

  my $is_prime = sub {
   my ( $number ) = @_;

   for ( 2 .. $number - 1 ) {
       return undef if $number % $_ == 0;
   }

   return 1;
 };

  while ( $limit > 0 ) {
        my $current = $perrin[ -2 ] + $perrin[ -3 ];
        elog( DEBUG, "Limit $n and current is $current" );
        push @perrin, $current;
        next if ! $is_prime->( $current );
        next if $seen->{ $current };

        # found!
        $seen->{ $current }++;
        return_next( $current );
        $limit--;
  }

return undef;
$CODE$
LANGUAGE plperl;


```
<br/>
<br/>

The workflow is the same as in the Raku implementation, but this time I use an hash to catch already seen Perrin prime numbers. Every time a new number is found (and it is tested as prime), it is appended to the result set. Once the requested number of primes is reached (i.e., `$limit` is zero), the function ends.
<br/>
It is worth saying that this can take a very long time!


<a name="task2plperl"></a>
## PWC 168 - Task 2 in PostgreSQL PL/Perl

A straightforward translation of the Raku implementation, where I use two inner subroutines to handle the test for prime-ness and gte the factors:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc168.task2_plperl( int )
RETURNS int
AS $CODE$

my ( $value ) = @_;

my $is_prime = sub {
   my ( $number ) = @_;

   for ( 2 .. $number - 1 ) {
       return 0 if ( $number % $_ == 0 );
   }

   return 1;
};

my $prime_factors = sub {
   my ( $number ) = @_;
   my @factors;

   return if $is_prime->( $number );

   for ( 2 .. $number - 1 ) {
       next if ! $is_prime->( $_ );
       next if $number % $_ != 0;
       next if $_ > $number;

       while ( ( $number % $_ ) == 0 ) {
             push @factors, $_;
             $number /= $_;
       }
   }

   return @factors;
};


my $value = join( '', $prime_factors->( $value ) );
while ( ! $is_prime->( $value ) ) {
      $value = join( '', $prime_factors->( $value ) );
}

return $value;

$CODE$
LANGUAGE plperl;
```
<br/>
<br/>


The workhorse of the function is the last four lines: `$value` is set to the join of its prime factors. Then, I do loop until a new prime number is found using the prime factors and re-join technique.



<a name="task1plpgsql"></a>
## PWC 168 - Task 1 in PostgreSQL PL/PgSQL

I implemented a function to test if a number is prime, then the workhorse function to generate the Perrin sequence, last a query to combine the two properties:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc168.is_prime( n bigint )
RETURNS bool
AS $CODE$
DECLARE
        i bigint;
BEGIN
        FOR i IN 2 .. n - 1 LOOP
            IF n % i = 0 THEN
               RETURN FALSE;
            END IF;
        END LOOP;

        RETURN TRUE;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc168.task1_plpgsql( l bigint default 5000 )
RETURNS SETOF BIGINT
AS $CODE$
DECLARE
        a bigint;
        b bigint;
        c bigint;
        d bigint;
BEGIN
        -- bootstrap
        a := 3;
        b := 0;
        c := 2;

        RETURN NEXT a;
        RETURN NEXT b;
        RETURN NEXT c;

        WHILE l > 0 LOOP
           d := a + b;
           a := b;
           b := c;
           c := d;

           RAISE INFO 'Level % value %', l, c;
           RETURN NEXT c;
           l := l - 1;
       END LOOP;


RETURN;
END
$CODE$
LANGUAGE plpgsql;


-- use more than 50 to get all the numbers
-- BUT THIS CAN BE VERY SLOW from 70 and beyond!
SELECT DISTINCT n
FROM pwc168.task1_plpgsql( 50 ) n
WHERE pwc168.is_prime( n )
ORDER BY 1
LIMIT 13;

```
<br/>
<br/>


The last query does all the work: limits the output result set, provides non-repeated numbers (via `DISTINCT` and extract only prime numbers via `is_prime`.


<a name="task2plpgsql"></a>
## PWC 168 - Task 2 in PostgreSQL PL/PgSQL

Exploits the same `is_prime()` function of the previous task, then add a `prime_factors()` function to get a result set with the prime factors of the given number.
<br/>
The task is implemented in the last function, that iterates over the result set of prime factors concatenating them together to a single string `v`, that is then converted into an integer and tested for primeness.  If not prime, the same set of oeprations is repeated.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc168.task2_prime_factors( n int )
RETURNS SETOF int
AS $CODE$
DECLARE
        i int;
        p bool;
BEGIN

        FOR i IN 2 .. n - 1 LOOP
            p := pwc168.is_prime( i );

            IF p AND n % i = 0  THEN
               WHILE n % i = 0 LOOP
                     n := n / i;
                     RETURN NEXT i;
               END LOOP;
            END IF;
        END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;


/*
testdb=> select * from pwc168.task2_plpgsql( 10 );
task2_plpgsql
---------------
773

*/

CREATE OR REPLACE FUNCTION
pwc168.task2_plpgsql( n int DEFAULT 10 )
RETURNS int
AS $CODE$
DECLARE
        i int;
        v text;
        p bool;
BEGIN
        v = '0';
        FOR i IN SELECT * FROM pwc168.task2_prime_factors( n ) LOOP
            v := v || i;
        END LOOP;


        p := pwc168.is_prime( v::int );

        WHILE NOT p LOOP
                i := v::int;
                v = '0';
                FOR i IN SELECT * FROM pwc168.task2_prime_factors( i ) LOOP
                    v := v || i;
                END LOOP;
                p := pwc168.is_prime( v::int );
        END LOOP;

        RETURN v::int;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
