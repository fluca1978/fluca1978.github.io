---
layout: post
title:  "Invoking (your own) Perl from PL/Perl"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perl
permalink: /:year/:month/:day/:title.html
---
A glance at how to invoke Perl code within PL/Perl code.

# Invoking (your own) Perl from PL/Perl

Invoking your own Perl code from PL/Perl, how hard can it be?
<br/>
Well, it turns out that it can be harder than you think.
PL/Perl is made to allow Perl interacting with the SQL world. Assume a function, named `get_prime` requires to invoke another Perl function `is_prime` to test if a number is prime or not.
<br/>
How is it possible to chain the function invocation?


## Invoking PL/Perl from PL/Perl via a query

One obvious possibility is to wrap `is_prime` into a PL/Perl function.
Since a PL/Perl function is, well, an ordinary function, it is always possible to call it *the SQL way*, as another ordinary function.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION is_prime( int )
RETURNS bool AS $CODE$
  my ( $n ) = @_;
  for ( 2 .. $n ) {
    last if $_ >= ( $n / 2 ) + 1;
    return 0 if $n % $_ == 0;
  }
  return 1;
$CODE$ LANGUAGE plperl;

CREATE OR REPLACE FUNCTION get_primes( int )
RETURNS SETOF int AS $CODE$
  for ( 1 .. $_[ 0 ] ) {
    my $result_set = spi_exec_query( "SELECT is_prime( $_ )" );
    return_next( $_ ) if ( $result_set->{ rows }[0]->{ is_prime } eq 't' );
  }
  return undef;
$CODE$ LANGUAGE plperl;
```
<br/>
<br/>


The function `get_primes` builds a query (`SELECT is_prime( $_ );`) that is executed several times in order to get the result.
<br/>
Advantages of this approach are that this is the natural way to query PostgreSQL functions, and therefore it would be possible to mix and match PL/Perl functions with other PL-functions. The main drawback is that this approach is tedious and error prone, since there is the need to build SQL queries. Moreover, handling invocation and argument passing will slow down the execution of the main function.


## Using anonymous subroutines

Luckily, Perl allows the definition of a subroutine within another subroutine, and to call it when required.
One way to achieve this is by *code references*.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION get_prime( int )
RETURNS SETOF int AS $CODE$
  my $is_prime = sub {
	 my ( $n ) = @_;
	 for ( 2 .. $n ) {
	   last if $_ >= ( $n / 2 ) + 1;
	   return 0 if $n % $_ == 0;
	 }
	 return 1;
  };

  for ( 1 .. $_[ 0 ] ) {
    return_next( $_ ) if $is_prime->( $_ );
  }

  return undef;
$CODE$ LANGUAGE plperl;
```
<br/>
<br/>

This makes *very easy and natural* to invoke Perl code (in this example, the `$is_prime` function) within PL/Perl.
The main advantage of this approach is that it is all written in pure Perl. The main drawback is that the `$is_prime` function is now *private* to the scope of the `get_prime` function, and therefore cannot be reused by other functions.


## Injecting a line of code at PL/Perl boot

PostgreSQL provides a set of `plperl` GUCs that allow you to set different properties of the Perl enrivornment.
One possibility, is to pre-declare a function so that once the PL/Perl engine will run, the function will be there.
Unluckily, GUCs do not allow for a setting to be split on different lines. Luckily, Perl is not Python, so you can write your own code in a single line.

<br/>
<br/>
```shell
# in postgresql.conf
plperl.on_plperl_init = 'sub is_prime { ...  }'
```
<br/>
<br/>

Therefore, when the PL/Perl engine starts, it gets the `is_prime` sub defined for free. This means that `get_rpimes` can be simply written as:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION get_primes( int )
RETURNS SETOF int AS $CODE$
   return [ grep { is_prime( $_ ) } ( 2 .. $_[ 0 ] ) ];
$CODE$ LANGUAGE plperl;
```
<br/>
<br/>

The main advantage of this approach is that it is simple. The main drawback is that it is complex, too.
Writing code within a single line is not a good habit. Most notably, this makes the code declared into `plperl.on_plperl_init` available to every instance of the PL/Perl engine, and this is a security risk!



## Injecting a module at PL/Perl boot

Following a similar approach, it is possible to place your custom code into a module and make PL/Perl `use` such module before the execution starts.
The first step required is to provide a Perl module and place it where PostgreSQL can find on the filesystem.

<br/>
<br/>
```perl
# $PGDATA/conf.d/fluca1978.pm
sub is_prime {
	 my ( $n ) = @_;
	 for ( 2 .. $n ) {
	   last if $_ >= ( $n / 2 ) + 1;
	   return 0 if $n % $_ == 0;
	 }
	 return 1;
}

1;
```
<br/>
<br/>

Now, it is possible to load the module either in `plperl.on_init` or in `plperl.on_plperlu_init`:

<br/>
<br/>
```shell
plperl.on_init = 'use lib q{/postgres/15/data/conf.f/fluca1978.pm}; use fluca1978;
```
<br/>
<br/>


Last, since the module has been loaded for every PL/Perl engine, the `get_primes` function remains as simple as:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION get_primes( int )
RETURNS SETOF int AS $CODE$
  return [ grep { is_prime( $_ ) } ( 2 .. $_[ 0 ] ) ];
$CODE$ LANGUAGE plperl;
```
<br/>
<br/>


In the case `plperl.on_init` does not include the `use` statement, the function `get_primes` should have been defined as `plperlu` loading the module itself.

The main advantage of this approach is that it provides modularity of available code.
The main drawback, as for the previous approach, is that it injects code into every PL/Perl engine that is going to be started.


## Using shared code

PL/Perl provides the `%_SHARED` hash that is shared among functions running within the same connections and with the same user.
This allows for storing an anonymous subroutine into the `%_SHARED` object and use it later.

The first step is to *initialize* the hash with the anonymous subroutine:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION my_plperl_init()
RETURNS VOID AS $CODE$
  $_SHARED{ is_prime } = sub {
	     my ( $n ) = @_;
	     for ( 2 .. $n ) {
	       last if $_ >= ( $n / 2 ) + 1;
	       return 0 if $n % $_ == 0;
	     }
	     return 1;
  };
$CODE$ LANGUAGE plperl;
```
<br/>
<br/>

Then, it is possible to make `get_primes` to use the shared reference:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION get_primes( int )
RETURNS SETOF int AS $CODE$
   return [ grep { $_SHARED{ is_prime }->( $_ ) } ( 2 .. $_[ 0 ] ) ];
$CODE$ LANGUAGE plperl;
```
<br/>
<br/>

The main adavantage of this approach is that it does not require a full code injection: only sessions that require the code to be shared will use it. The main drawback is that there is no *real sharing* of code, it is just temporary code living in the session space. Moreover, this approach requires an initialization phase, that can be error prone.


# Conclusions

Being PL/Perl, well, Perl, *there's more than one way to do it!*
<br/>
Depending on the aims and constraints, there are different ways to invoke Perl code from other PL/Perl code. The main considerations, when choosing an approach or another, are related to **code reusability** and **performances**. Wrapping Perl into PL/Perl provides for better code reusability, but requires more resources and code bloating. Using a pure Perl approach provides for the best performances and code readibility, but can open the door to some security risks.
