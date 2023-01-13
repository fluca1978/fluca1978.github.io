---
layout: post
title:  "Perl Weekly Challenge 159: numbers and numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 159: numbers and numbers

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 159](https://perlweeklychallenge.org/blog/perl-weekly-challenge-159/){:target="_blank"}.

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
## PWC 159 - Task 1

It has been harder to understand what to implement than to implement it!
The task was about producing a Farey sequence, that is a sequence of fractions in increasing order of value, starting from `0/1` to `1/1` and with unique values.




<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {
    my $start = 0/1.Rat;
    my $end   = 1/1.Rat;

    my @farey = $start, $end;

    for 2 ..  $n -> $denominator {
        @farey.push( |( 1 .. $denominator ).map: * / $denominator );
    }

    @farey.unique.sort.map(  *.nude.join( '/' ) ).say;
}

 ```
<br/>
<br/>

The `for` loop generates all the fractions for any order, and that I do extract all `unique` numbers, sort them and `map` to their `nude` representation that is an array of numerator and denominator that are then joined by `/`.

<a name="task2"></a>
## PWC 159 - Task 2

A Mobius number is a number within `0,1,-1` depending on the size of the list of its prime numbers and the free square roots.

<br/>
<br/>
```raku
sub prime-factors( Int $n ) {
    my $number = $n;
    my @factors;

    my $factor = 2;
    while ( $number > 1 && $factor <= $number ) {
        if ( $number %% $factor ) {
            @factors.push: $factor;
            $number /= $factor;
        }
        else {
            $factor++;
        }
    }

    return @factors;
}


sub MAIN( Int $n where { $n > 0 } ) {

    my @prime-factors = prime-factors( $n );
    '0'.say and exit if @prime-factors.elems != @prime-factors.unique.elems;
    '1'.say and exit if @prime-factors.unique.elems %% 2;
    '-1'.say and exit;

}

```
<br/>
<br/>


The function `prime-factors` computes the prime factors array for the given number. Then it is just a matter to see the length of such list compared against the unique list of such factors.


<a name="task1plperl"></a>
## PWC 159 - Task 1 in PostgreSQL PL/Perl


An implementation that relies only on Perl operators and that returns a list of text representing the fractions.
I use an hash, `%farey`, indexed by a fraction and that has a value representing the fraction as a string.
Note that I do simplify every fraction to its minimum term before inserting into the hash.
<br/>
Next I do extract information about occurencies of a term, that is `%unique_counter` contains the fraction as a key and the occurencies as a value. The first time a fraction is encountered, it is appended to the result set, then it is skipped.
<br/>
Therefore, the function can return the values (as text) of every unique fraction.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc159.farey( int )
RETURNS SETOF text
AS $CODE$
   my ($n) = @_;

   my %farey;

   for my $denominator ( 2 .. $n ) {
       for my $number ( 1 .. $denominator ) {

           # reduce things like 2/4 to 1/2
          ( $denominator, $number ) /= $number       if ( $denominator % $number == 0 );
          ( $denominator, $number ) /= $denominator  if ( $number % $denominator == 0 );

           $farey{ $number/$denominator } = "$number/$denominator";
       }
   }

   # bootstrap
   return_next( '0/1' );

   my %unique_counter;
   for my $key ( sort keys( %farey ) ) {
       # ensure only one item is printed out
       $unique_counter{ $key }++;
       next if $unique_counter{ $key } > 1;
       next if $key == 1;  # last term in the sequence

       return_next( $farey{ $key } );
   }

   # end term
   return_next( '1/1' );
   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 159 - Task 2 in PostgreSQL Pl/Perl

An implementation similar to the Raku one, but in pure Perl.
<br/>
The `$prime_factors` is an anonymous function to compute the hash of prime factors of the given number. The hash is indexed by the factor and has the occurencies as value. Then I do convert the hash into an array `@unique_prime_factors` by iterating on the keys of the previous hash, and counting also the total amount of terms in the factorization.
<br/>
Last it is only a matter of understanding the size of the `@unique_orime_factors` list.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc159.mobius( int )
RETURNS int
AS $CODE$

   my ( $n ) = @_;

   # a routine to compute the prime
   # factors of the given number
   my $prime_factors = sub {
      my ( $number ) = @_;
      my %factors;

      my $factor = 2;
      while (  $number > 1 && $factor <= $number ) {
            if ( $number % $factor == 0 ) {
               $factors{ $factor }++;
               $number /= $factor;
            }
            else {
                 $factor++;
            }
      }

      return %factors;
   };


   my %prime_factors = $prime_factors->( $n );

   # to get the unique prime factors I have to "count"
   # them only once per key
   my @unique_prime_factors;
   my $occurrencies_prime_factors = 0;
   for ( keys %prime_factors ) {
       push @unique_prime_factors, $_;
       $occurrencies_prime_factors += $prime_factors{ $_ };
   }


   return 0 if @unique_prime_factors != $occurrencies_prime_factors;
   return 1 if @unique_prime_factors % 2 == 0;
   return -1;


$CODE$
LANGUAGE plperl;


```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 159 - Task 1 in PostgreSQL Pl/PgSQL

This implementation is done in two steps:
- the `farey_not_unique` function provides a set of not-unique values, where `f` is the text representation of the fraction, and `v` is the value computed of the fraction;
- the SQL query does a materialization of the `order by` value query and provides the distinct set of fractions.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc159.farey_not_unique( n int )
RETURNS TABLE( f text, v numeric )
AS $CODE$
DECLARE
     numerator   int;
     denominator int;
     dd          int;
     nn          int;
BEGIN

        -- bootstrap term
        SELECT '0/1', 0
        INTO f, v;

        RETURN NEXT;


        FOR denominator IN 2 .. n  LOOP
            FOR numerator IN  1 .. denominator  LOOP
                nn := numerator;
                dd := denominator;

                IF dd % nn = 0 THEN
                   dd := dd / nn;
                   nn := 1;
                END IF;

                IF nn % dd = 0 THEN
                   nn := nn / dd;
                   dd := 1;
                END IF;

                IF nn / dd = 1 THEN
                   CONTINUE;
                END IF;

                SELECT nn || '/' || dd, nn/dd::numeric
                INTO  f, v;

                RETURN NEXT;

            END LOOP;
        END LOOP;

        -- end term
        SELECT '1/1', 1
        INTO f, v;

        RETURN NEXT;

RETURN;
END
$CODE$
LANGUAGE plpgsql;


WITH farey AS (
     SELECT distinct( f ), v
     FROM pwc159.farey_not_unique( 5 )
     ORDER BY v
)
SELECT f
FROM farey;


```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 159 - Task 2 in PostgreSQL Pl/PgSQL

Here there's a `prime_factors` function that returns the list, as integers, of the prime factors of a given number.
The SQL query then counts the number of prime factors and the number of unique factors, exported as `cc` and `c` respectively. Last, a `CASE` statement does the trick of converting the number of prime factors into a `1` or `-1` or `0'.
<br/>
Note the usage of a `psql` variable `:n` that is used to set the value to use within the query, by means of a `\set` directive.
That's `psql` code, so in order to use the same query into another client, please substitute `:n` with the value you want to test.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc159.prime_factors( n int )
RETURNS SETOF int
AS $CODE$
DECLARE
        factor int;
BEGIN
        factor := 2;

        WHILE ( factor <= n AND n > 1 ) LOOP
              IF n % factor = 0 THEN
                 n := n / factor;
                 RETURN NEXT factor;
              ELSE
                factor := factor + 1;
              END IF;
        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;


\set n 5

WITH count_prime_factors(  c, cc ) AS
(
        SELECT count( distinct pf ), count( pf )
        FROM pwc159.prime_factors( :n ) pf
)
SELECT :n AS number,
       CASE
       WHEN c - cc <> 0 THEN 0
       WHEN c % 2  = 0  THEN 1
       ELSE -1
       END

FROM count_prime_factors;


```
<br/>
<br/>
