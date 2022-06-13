---
layout: post
title:  "Perl Weekly Challenge 169: primes, primes, and more primes!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 169: primes, primes and more primes!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 169](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0169/){:target="_blank"}.

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
## PWC 169 - Task 1

Compute first twenty *brilliant* numbers, those with exactly two factors of the very same length:




<br/>
<br/>
```raku
sub is-brilliant( $num ) {
    my $number = $num;

    # get all the factors
    my @factors;

    for 2  ..^ $number {
        next if ! $_.is-prime;

        while ( $number %% $_ ) {
            @factors.push: $_;
            $number /= $_;
        }
    }

    return False if @factors.elems != 2;
    return False if @factors[0] == @factors[1];
    return True if @factors[0].Str.chars == @factors[1].Str.chars;
}


sub MAIN( Int $limit where { $limit > 0 } = 20 ) {

    my @brilliant = lazy gather {
      take $_ if is-brilliant( $_ ) for 1 .. Inf;
    };

    @brilliant[ 0 .. $limit ].join( "\n" ).say;

}

 ```
<br/>
<br/>

The `is-brilliant` function does all the work. Given a number, it extracts its factors, and then checks if the number has exactly two factors with the same character length.
<br/>
The `MAIN` does use a `lazy gather` to extract the values.

<a name="task2"></a>
## PWC 169 - Task 2

Compute Achille's numbers, those that are powerful but imperfect.

<br/>
<br/>
```raku
sub compute-factors( $num ) {
    my $number = $num;
    my @factors;

    for 2  ..^ $number {
        next if ! $_.is-prime;

        while ( $number %% $_ ) {
            @factors.push: $_;
            $number /= $_;
        }
    }

    return @factors;
}


sub is-achille( $num )
{
    my Bag $bag = Bag.new: compute-factors( $num );
    return False if $bag.keys.elems <= 1;
    return $bag.values.min >= 2 && ( [gcd] $bag.values ) == 1;
}

sub MAIN( Int $limit where { $limit > 0 } = 20 ) {

    my @achille = lazy gather {
      take $_ if is-achille( $_ ) for 1 .. Inf;
    };

    @achille[ 0 .. $limit ].join( "\n" ).say;

}


```
<br/>
<br/>

I used a `Bag`, to store the factors of a given number and its occurrencies. Thanks to the presence of the `gcd` list operator, it becomes really easy to implement the `is-achille` function.
<br/>
The `MAIN` uses a `lazy gather` to extract all the values.


<a name="task1plperl"></a>
## PWC 169 - Task 1 in PostgreSQL PL/Perl


A translation of the Raku implementation. The boring part is that all the pieces have to be provided as anoymous code blocks to check if a number is prime and to get the factors:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc169.task1_plperl( int )
RETURNS SETOF int
AS $CODE$
  my ($limit) = @_;

  my $is_prime = sub {
     my ($value) = @_;

     for ( 2 .. $value - 1 ) {
         return 0 if $value % $_ == 0;
     }

     return 1;
  };

  my $compute_factors = sub {
     my ($value) = @_;
     my @factors;

     for ( 2 .. $value - 1 ) {
         next if ! $is_prime->( $_ );

         while ( $value % $_ == 0 ) {
               push @factors, $_;
               $value /= $_;
         }
     }

     return @factors;
  };


  for ( 1 .. 9999999 ) {
      my @factors = $compute_factors->( $_ );
      elog( DEBUG, "Number $_ with factors " . join(',', @factors) );
      next if ( @factors != 2 );
      next if $factors[ 0 ] == $factors[ 1 ];
      if ( length( $factors[ 0 ] ) == length( $factors[ 1 ] ) ) {
         $limit--;
         return_next( $_ );
      }

      last if ! $limit;
  }

  return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 169 - Task 2 in PostgreSQL PL/Perl

Again, a translation of the Raku solution. This time however, I decided to use a module to take advantage of `gcd` for a list, so the language has changed to `plperlu` to allow the system to load the `Math::BigInt` module:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc169.task2_plperl( int )
RETURNS SETOF int
AS $CODE$
   use Math::BigInt;


   my ($limit) = @_;

  my $is_prime = sub {
     my ($value) = @_;

     for ( 2 .. $value - 1 ) {
         return 0 if $value % $_ == 0;
     }

     return 1;
  };

  my $compute_factors = sub {
     my ($value) = @_;
     my @factors;

     for ( 2 .. $value - 1 ) {
         next if ! $is_prime->( $_ );

         while ( $value % $_ == 0 ) {
               push @factors, $_;
               $value /= $_;
         }
     }

     return @factors;
  };

  my $min = sub {
     my $found = shift @_;
     for ( @_ ) {
         $found = $_ if $_ < $found;
     }

     return $found;
  };

  my $is_achille = sub {
     my ($number) = @_;
     my $bag = {};

     for ( $compute_factors->( $number ) ) {
         $bag->{ $_ }++;
     }

    return $min->( values( %$bag ) ) >= 2 && Math::BigInt::bgcd( values( %$bag ) )->numify == 1;
  };


  for ( 1 .. 999999 ) {
    if ( $is_achille->( $_ ) ) {
       $limit--;
       return_next( $_ );
    }

    last if ! $limit;
  }

  return undef;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


I could have used other modules to get prime numbers and factors, but I preferred to keep as simpler as possible the solution.
<br/>
Here implementing the `Bag` class behavior is easy thanks to autovivification!


<a name="task1plpgsql"></a>
## PWC 169 - Task 1 in PostgreSQL PL/PgSQL

Here I split the solution into different functions, that can provide the `is_prime` and `compute_factors` capabilities:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc169.compute_factors( n int )
RETURNS SETOF INT
AS $CODE$
DECLARE
        i int;
BEGIN

        FOR i IN 2 .. ( n - 1 ) LOOP
            IF NOT pwc169.is_prime( i ) THEN
               CONTINUE;
            END IF;

            WHILE n % i = 0 LOOP
                  RETURN NEXT i;
                  n = n / i;
            END LOOP;
        END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc169.task1_plpgsql( n int DEFAULT 20 )
RETURNS SETOF INT
AS $CODE$
DECLARE
        i int;
        f int;
        current_length int := 0;
        current_count  int := 0;
        ok bool := false;
        previous_f     int := 0;
BEGIN

        FOR i IN 2 .. 100000 LOOP

            current_length := 0;
            current_count  := 0;
            ok             := true;
            previous_f     := 0;

            FOR f IN SELECT * FROM pwc169.compute_factors( i ) ORDER BY 1 LOOP

                current_count := current_count + 1;
                IF current_length = 0 THEN
                   current_length := length( f::text );
                END IF;

                IF length( f::text ) <> current_length OR current_count > 2 OR f = previous_f THEN
                   ok := false;
                   EXIT;
                END IF;

                previous_f := f;
            END LOOP;

            IF ok AND previous_f <> 0 THEN

               RETURN NEXT i;
               IF n = 0 THEN
                  RETURN;
               END IF;
               n := n - 1;
            END IF;
        END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The main loop within the `task1_plpgsql()` function iterates over all numbers, and then inner-interates on factors. For each factor I see if I've reached more than two factors, or if the same factor is present, so that I can quickly exit. Otherwise, if the `text` representation of the numbers has the same length, I can spurt the number appending it to the result set.



<a name="task2plpgsql"></a>
## PWC 169 - Task 2 in PostgreSQL PL/PgSQL

Exploiting the same `is_prime` and `compute_factors` functions from the previous task, and the PostgreSQL inner `gcd` (that works on a couple of numbers at a time), the code can be implemented as follows:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc169.task2_plpgsql( n int DEFAULT 20 )
RETURNS SETOF INT
AS $CODE$
DECLARE
        i int;
        current_min int;
        current_gcd int;
        previous_gcd int;
BEGIN

        FOR i IN 1 .. 999999 LOOP

           WITH bag AS (
                SELECT f, count(*) AS counter
                FROM pwc169.compute_factors( i ) f
                GROUP BY f
           )
           SELECT min( counter )
           INTO current_min
           FROM bag
           ;

           IF current_min < 2 THEN
              CONTINUE;
           END IF;

           previous_gcd := -1;
           FOR current_gcd IN  SELECT count(f) FROM pwc169.compute_factors( i ) f GROUP BY f LOOP
               IF previous_gcd < 0 THEN
                  previous_gcd := current_gcd;
                  CONTINUE;
               END IF;

               previous_gcd := gcd( previous_gcd, current_gcd );
           END LOOP;

           IF previous_gcd = 1 THEN
              RETURN NEXT i;
              IF n = 0 THEN
                 RETURN;
              END IF;
              n := n - 1;
           END IF;
        END LOOP;
RETURN;
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>

Note the inner query that exploits a CTE named `bag` in honor of the same `Bag` class used in the Raku solution.
