---
layout: post
title:  "Perl Weekly Challenge 171: numbers and references"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 171: numbers and refenreces

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 171](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0171/){:target="_blank"}.

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
## PWC 171 - Task 1

This task was about to compute the first 20 odd abundant numbers, those that have a sum of divisors greater than the number itself.

<br/>
<br/>
```raku
sub is-abundant( Int:D $n ) {
    my @divisors = 1;
    @divisors.push: $_ if $n %% $_ for ( 2 .. $n / 2 );
    return @divisors.sum > $n;
}

sub MAIN() {
    my @abundant = lazy gather {
      take $_ if $_ !%% 2 && is-abundant( $_ ) for ( 1 .. Inf );
    };

    @abundant[ 0 .. 20 ].join( "\n" ).say;
}

 ```
<br/>
<br/>

I decided to create an `is-abundant` function that extracts all the divisors, put them int the `@divisors` array and sum them, returning if the sum if greater or not than the original number.
<br/>
Then, in the `MAIN`, I `lazy gather` over all numbers and provide only those odd and that are abundant.

<a name="task2"></a>
## PWC 171 - Task 2

*I'm not sure I've correctly understood the task.*
The task was requiring you to make a function that accepts two functions and returns the result of invoking them in a nested way.

<br/>
<br/>
```raku
sub compose( Sub:D $f, Sub:D $g ) {
    return sub (*@a) {
        "F( G( { @a } ) )".say;
        $f( $g( @a ) )
    };
}

sub MAIN() {
    my $f = sub (*@a) {
        "F( { @a } )".say;
        return @a;
    };
    my $g = sub (*@a) {
        "G( { @a } )".say;
        return @a;
    };

    my $h = compose( $f, $g );
    $h( 'PWC 171' );
}


```
<br/>
<br/>


I produced two function, `$f` and `$g` that both slurp their arguments and return them unmodified. Of course, in real like, such functions are going to do something.
Then `compose` accepts two defined `Sub` references and invokes them in a nested way.


<a name="task1plperl"></a>
## PWC 171 - Task 1 in PostgreSQL PL/Perl

A translation of the Raku solution:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc171.task1_plperl( int )
RETURNS SETOF INT
AS $CODE$
my ($limit) = @_;

my $is_abundant = sub {
   my ( $n ) = @_;
   my @divisors = ( 1 );

   for ( 2 .. $n / 2 ) {
       next if $n % $_ != 0;
       push @divisors, $_;
   }

   my $sum = 0;
   $sum += $_ for ( @divisors );

   return $sum > $n;
};


for ( 1 .. 99999 ) {
    next if $_ % 2 == 0;
    if ( $is_abundant->( $_ ) ) {
       $limit--;
       return_next( $_ );
    }

    last if $limit <= 0;
}

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `$is_abundant` code references implements the function that sums all divisors of the given odd number and returns true if the number is abundant.
Then, the main loop does a `return_next` to append a new number to the result set in the case it is abundant.



<a name="task2plperl"></a>
## PWC 171 - Task 2 in PostgreSQL PL/Perl

Assuming I understood the tast, theimplementation is just a *crafted `SELECT`*:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc171.task2_f( text )
RETURNS TEXT
AS $CODE$
   elog( DEBUG, "F( $_[0] )" );
   return $_[0];
$CODE$
LANGUAGE plperl;


CREATE OR REPLACE FUNCTION
pwc171.task2_g( text )
RETURNS TEXT
AS $CODE$
   elog( DEBUG, "G( $_[0] )" );
   return $_[0];
$CODE$
LANGUAGE plperl;


/*
testdb=> select * from pwc171.task2_plperl( 'pwc171.task2_f', 'pwc171.task2_g', 'Hello World' );
DEBUG:  Query [SELECT pwc171.task2_f( pwc171.task2_g( 'Hello World' ) ) AS compose;]
DEBUG:  G( Hello World )
DEBUG:  F( Hello World )
 task2_plperl
--------------
 Hello World
*/
CREATE OR REPLACE FUNCTION
pwc171.task2_plperl( text, text, text )
RETURNS TEXT
AS $CODE$



my $compose = sub {
   my $query = sprintf( "SELECT %s( %s( '%s' ) ) AS compose;",
               $_[ 0 ],
               $_[ 1 ],
               $_[ 2 ] );
   elog( DEBUG, "Query [$query]" );
   my $result_set = spi_exec_query( $query );
   return $result_set->{ rows }[ 0 ]->{ compose };
};

return $compose->( @_ );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I wrote a couple of functions, `f` and `g` just to use to test the machinery.
<br/>
The main function build a crafted nested `SELECT` that invokes all the functions in the right order, and then returns the result outside. Why using an anonymous code reference? Well, because if you want to transform that into a set returning function is easier!


<a name="task1plpgsql"></a>
## PWC 171 - Task 1 in PostgreSQL PL/PgSQL

Quite same approach as in the PL/Perl solution:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc171.task1_plpgsql( l int DEFAULT 20 )
RETURNS SETOF INT
AS $CODE$
DECLARE
        s int := 0;
        i int;
        d int;
BEGIN
      FOR i IN  2 .. 99999  LOOP
          IF i % 2 = 0 THEN
             CONTINUE;
          END IF;
          s := 0;
          FOR d in  2 .. ( i / 2 )  LOOP
              IF i % d = 0 THEN
                 s := s + d;
              END IF;
          END LOOP;

          IF s > i THEN
             RETURN NEXT i;
             l := l - 1;

             IF l = 0 THEN
                RETURN;
             END IF;
          END IF;
    END LOOP;
RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The only different with the Perl and Raku solutions is that I don't care about divisors, but keep only the sum of them while processing them.



<a name="task2plpgsql"></a>
## PWC 171 - Task 2 in PostgreSQL PL/PgSQL

Easy enough to be done with a simple query:
<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc171.task2_plpgsql( f text DEFAULT 'pwc171.task2_f',
                      g text DEFAULT 'pwc171.task2_g',
                      v text DEFAULT 'PWC 171' )
RETURNS SETOF TEXT
AS $CODE$
BEGIN
        RETURN QUERY
        EXECUTE format( 'SELECT * FROM %s( %s( $$%s$$ ) )', f, g, v );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The `EXECUTE` accepts a dynamically built query, built on top of `format()` (a kind of `printf`) that does the `SELECT` of the nested functions. Note the usage of *dollar quoting* `$$` around the function argument to escape it in already quoted string.
