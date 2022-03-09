---
layout: post
title:  "Perl code reuse in Pl/Perl thru pg_proc and anonymous code blocks"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perl
permalink: /:year/:month/:day/:title.html
---
Perl is great. PostgreSQL is great. And great plus great means super-powers!


# Perl code reuse in Pl/Perl thru pg_proc and anonymous code blocks

PostgreSQL allows you to write executable code, e.g., `FUNCTION`s and `PROCEDURE`s in Perl thru its extension language Pl/Perl (`plperl` and `plperlu`).
But sometimes there is the need to use the same Perl block over and over again across different code blocks and functions.
<br/>
There are different approaches, most notably the *module* one: abstract your behavior into a module and load it whenever you need. Yeah, this means using `plperlu`, but it is a fair tradeoff.
<br/>
<br/>
However, keeping in mind how PostgreSQL stores procedures and their code, it is possible to use a more fancy approach. In this post I show you a couple of simple examples as proofs of concept, clearly in order to push this into production there is the need for a more sophisticated approach.


## An easy function in Pl/Perl

Let's start simple: a Pl/Perl function to say prime numbers.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
fluca.is_prime( int )
RETURNS bool
AS $CODE$
   return 1 if $_[0] <= 2;

   for my $i ( 2 .. $_[0] - 1 ) {
       return 0 if $_[0] % $i == 0;
   }

   return 1;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Quite simple, uh? Now imagine that we need to prepare another function that needs to generate prime numbers, and thus needs to know if a given number is prime or not.
<br/>
One approach could be to call the above `fluca.is_prime()` function, but this will slow down the whole process. But after all, this is the building block logic on functions!
<br/>
Another approach could be to take apart the above bunch of Perl code, create a module, and use it wherever needed. But it is not the approach followed here.
<br/>
Again, another approach could be to store a block reference into the `%_SHARED` global hash.
<br/>
Last, why not querying the catalog `pg_proc` to extract the *source code* of the above function and wrap it into another Perl anonymous code block? It goes like this:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
fluca.generate_primes_up_to( int )
RETURNS SETOF int
AS $CODE$

my $query = "select prosrc from pg_proc where proname = 'is_prime' and pronamespace = ( select oid from pg_namespace where nspname = 'fluca' );";
my $code = spi_exec_query( $query, 1 )->{ rows }[ 0 ]->{ prosrc };

my $is_prime = eval( "sub { $code }; " );

for my $n ( 1 .. $_[0] ) {
    return_next( $n ) if $is_prime->( $n );
}

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `$query` statement selects the `pg_proc.prosrc` text field that contains the source code, whatever you have written between `$CODE$` separators. That's because `plperl` is a *pl* language, therefore its source code is stored in the system catalog.
<br/>
Having stated that, the `$code` string contains the block of code, so it was like the variable was declared as follows:


<br/>
<br/>

``` perl
my $code = " return 1 if $_[0] <= 2;

   for my $i ( 2 .. $_[0] - 1 ) {
       return 0 if $_[0] % $i == 0;
   }

   return 1;";
```
<br/>
<br/>

Seems like a Perl sub, but it is not (yet). There is the need to wrap the bunch of code into a `sub` declaration, and this is the easy part, and then we need to compile it. That's the task of `eval( "sub { $code };" )`, that creates an anonymous subroutine with the source code extracted from the other function.
<br/>
Such code is stored into a scalar `$is_prime` that is then used as a standard anonymous subroutine via `->`.
<br/>
And that's all!


### Advantages

The main advantage of the above approach is that whenever a change is done nto `fluca.is_prime()`, the same change is immediatly reflected into `fluca.generate_primes_up_to()`, because the source code of the former is always queried at the time the latter starts its execution.


### Drawbacks

Time!
<br/>
Extracting the code and compiling it every time requires time and resources, so it can be a pitfall for big Perl code blocks. There are different modules that can help in this scenario, e.g., `[Perl::Parse](https://metacpan.org/pod/Parse::Perl){:target="_blank"}`.
<br/>
An *hidden* drawback is that the two functions are not explicitly coupled, so if `fluca.is_prime` is accidentaly deleted, the other function will no more be able to run at all!


## The `SETOF` problem

Reusing a piece of code that returns a scalar is simple, but what about functions that return sets?
<br/>
Assume there is the need for a function that returns all the *even* numbers up to a limit, and does that efficiently, that is returning one value at a time.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
fluca.generate_evens( int )
RETURNS SETOF int
AS $CODE$
for ( 1 .. $_[0] ) {
  return_next( $_ ) if $_ % 2 == 0;
}

return undef;
$CODE$
LANGUAGE plperl;

```

<br/>
<br/>

While the function is really simple, the problem of the sets arises immediatly: Pl/Perl provides particular ways of interacting with PostgreSQL, and `return_next` is one of such ways. Long story short: `return_next` yelds the function adding a new element to the current result set.
<br/>
Since (regular) Perl does not have a `return_next` operator nor a function, how to translate such code? A very *inefficient* approach is to put the result set into an array and return the whole array. It is not the same as `return_next`, because there is no *yelding*, but it can work. Therefore, a function that wants to use the previous code could inject an array on the function prologue and substitute `return_next` with a regular array returning.
<br/>
Imagine we want to build a function that computes odd numbers on top of the even ones; the code looks like the following snippet.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
fluca.generate_odds( int )
RETURNS SETOF int
AS $CODE$
my $query = "select prosrc from pg_proc where proname = 'generate_evens' and pronamespace = ( select oid from pg_namespace where nspname = 'fluca' );";
my $code = spi_exec_query( $query, 1 )->{ rows }[ 0 ]->{ prosrc };

$code = "my \@return_values;\n" . $code ;
$code =~ s/return_next\s*\(/push( \@return_values,/g;
$code =~ s/return\s+undef\.*;/return \@return_values;/g;

my $generate_evens = eval( "sub { $code }; " );
my @odds = map( { $_ + 1 } $generate_evens->( 10 ) );

for ( @odds ) {
  return_next( $_ );
}

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The base idea is the same as in the previous case: query `pg_proc` to get the source of the function and store it into `$code`.
<br/>
Then, add the declaration for an array, named `@return_values` (a better and unique name should be chosen), and substitue with a regular expression all `return_next` with a `push` into the above array, removing also any `return undef` (that in PlPerl is the way to end the result set).
<br/>
Yeah, I hear you screaming! This is surely something not to do in production, but it is a simple and dirty way to make Perl do what you want.
<br/>
As in the previous case, store the result of `eval`uating the so rewritten `$code` into a scalar named `$generate_evens` and use it as you prefer.
<br/>
<br/>
**Danger Will Robinson!**
<br/>
The regex substitution is awful because it will go beyond the *scope* where `return_next` applies. Imagine to apply the same technique recursively to `fluca.generate_odds()`: there is a `return_next` level inside the code extracted from `pg_proc` and an outer scope with `return_next` used within the function itself. The regular expression is not able to find out the scope, so both `return_next` will appear similar and will get substituted in the very same manner. **And that's why you should not use such approach in production**! Again, there are Perl modules to get rid of these details and get things done in the right way.

# Conclusions

Perl is great. It allows you to build dynamic code in a very dynamic way.
<br/>
PostgreSQL is great. It allows you to inspect every single part of the system, including executable code.
<br/>
The Perl power to push some code out of a PostgreSQL table (`pg_proc`) into a scalar, so to use it later on, allows for code sharing among Pl/Perl functions and routines.
<br/>
It is up to you to decide to shoot yourself in the foot or hit something valuable!
