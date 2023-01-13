---
layout: post
title:  "Perl Weekly Challenge 170: primordial matrix!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 170: primordial matrix!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 170](https://perlweeklychallenge.org/blog/perl-weekly-challenge-170/){:target="_blank"}.

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
## PWC 170 - Task 1

Compute the first ten *primordial* numbers, those that are made up of multiplication of prime numbers.
This is a piece of cake in Raku:




<br/>
<br/>
```raku
sub MAIN( Int $limit = 10 ) {

    my @primes;
    my @primordial = lazy gather {
        for ( 1 .. Inf ) {
            next if ! .is-prime;
            @primes.push: $_;
            take [*] @primes;
        }
    };

    @primordial[ 0 .. $limit ].join( "\n" ).say;
}

 ```
<br/>
<br/>

There is a `lazy-gather` that populates the `@primes` array and returns the reduction of the multiplication of all its values.
<br/>
In order to speed up the computation, once could also cache the result of the last computation and multiply it by the next prime in order to not consume both space and time.

<a name="task2"></a>
## PWC 170 - Task 2

A strange matrix product, named Kronecker product.
Assuming both the matrixes have the same shape, this is also another piece of cake in Raku:

<br/>
<br/>
```raku
sub MAIN() {

    my @a = [1, 2], [3, 4];
    my @b = [5, 6], [7, 8];

    my @result;

    for 0 ..^ @a.elems -> $row_a {
        for 0 ..^ @b.elems -> $row_b {
            @result.push: [ @a[ $row_a ].List X* @b[ $row_b ].List ];
        }
    }

    @result.join( "\n" ).say;

}


```
<br/>
<br/>

I extract every *line* out of a matrix and multiply it with the corresponding line on the other matrix, bia the `X*` cross-multiply operator. The result is placed into the `@result` matrix.


<a name="task1plperl"></a>
## PWC 170 - Task 1 in PostgreSQL PL/Perl

Very likely the Raku solution:

<br/>
<br/>

``` sql
create table if not exists a( a int, b int );
truncate table a;
insert into a values (1,2), (3,4);
create table if not exists b( a int, b int );
truncate table b;
insert into b values (5,6), (7,8);

CREATE SCHEMA IF NOT EXISTS pwc170;

CREATE OR REPLACE FUNCTION
pwc170.task2_plperl( text, text )
RETURNS TABLE( a int, b int, c int, d int )
AS $CODE$
   my ( $table_a, $table_b ) = @_;
   elog( DEBUG, "Reading tables $table_a and $table_b" );

   my ( $rs_a, $rs_b );
   $rs_a = spi_exec_query( "SELECT a,b FROM $table_a" );
   $rs_b = spi_exec_query( "SELECT a,b FROM $table_b" );

   for my $row_a ( 0 .. $rs_a->{ processed } - 1 ) {
      elog( DEBUG, "Loop A $row_a out of " . $rs_a->{ processed } );
      my ($aa, $ab) = ( $rs_a->{ rows }[ $row_a ]->{ a }, $rs_a->{ rows }[ $row_a ]->{ b } );

      for my $row_b ( 0 .. $rs_b->{ processed } - 1 ) {
        elog( DEBUG, "Loop B $row_b out of " . $rs_b->{ processed } );
        my ($ba, $bb) = ( $rs_b->{ rows }[ $row_b ]->{ a }, $rs_b->{ rows }[ $row_b ]->{ b } );

        elog( DEBUG, "Computing $aa $ab X* $ba $bb" );
        my $result = {
         a => $aa * $ba,
         b => $aa * $bb,
         c => $ab * $ba,
         d => $ab * $bb,
        };
        return_next( $result );
      }
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Tables `a` and `b` are initialized with the correct values.
<br/>
The function takes as input arguments the names of the table to query, and do a nested loop on every single row extracted from each table. Columns are assigned to four different variables that are then multiplied togheter and stored into an hash that plays the role of the output table. Then the result is appended to the result set.



<a name="task2plperl"></a>
## PWC 170 - Task 2 in PostgreSQL PL/Perl

A little more complicated solution, that is based on the fact that the matrixes are now tables:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc170.task2_plperl( int )
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
## PWC 170 - Task 1 in PostgreSQL PL/PgSQL

This approach does the *caching* I was writing about in the Raku implementation:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc170.is_prime( v bigint )
RETURNS bool
AS $code$
DECLARE
        i bigint;
BEGIN
        FOR i IN  2 .. v - 1  LOOP
            IF v % i = 0 THEN
               RETURN false;
            END IF;
        END LOOP;

        RETURN TRUE;
END
$code$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc170.task1_plpgsql( l int default 10 )
RETURNS SETOF INT
AS $CODE$
DECLARE
        v bigint := 1;
        i bigint;
BEGIN
        FOR i IN SELECT n FROM generate_series( 1, 100000 ) n LOOP
            IF pwc170.is_prime( i ) THEN
               v := v * i;
               l := l - 1;
               RETURN NEXT v;
            END IF;

            IF l <= 0 THEN
               RETURN;
            END IF;
        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

There is a `is_prime` utility function, but in the main loop I do keep multiplying the previous result by the new computed prime, returning it into the result set. Therefore, there is no *array of primes* to iterate onto.



<a name="task2plpgsql"></a>
## PWC 170 - Task 2 in PostgreSQL PL/PgSQL

SQL has already such computation: cross join!
<br/>
However, let's add something more useful on top of it: the capability to specify from which tables extract the resulting matrix. In this example I'm going to use the same tables `a` and `b` of the previous PL/Perl implementation:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc170.task2_plpgsql( ta text, tb text )
RETURNS TABLE ( a int, b int, c int, d int )
AS $CODE$
DECLARE
BEGIN
        RETURN QUERY
        EXECUTE format( 'SELECT a.a * b.a, a.a * b.b, a.b * b.a, a.b * b.b FROM %I a, %I b',
                        ta, tb );
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>

The function takes as input arguments the names of the tables to use as matrixes.
Then it formats a query that does the cross join with the correct names and returns the query as a whole.
