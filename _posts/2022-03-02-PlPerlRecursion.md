---
layout: post
title:  "Pl/Perl Recursion"
author: Luca Ferrari
tags:
- postgresql
- perlweeklychallenge
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Some thoughts and experiments in Pl/Perl recursion.

# Pl/Perl Recursion

While solving the [Perl Weekly Challenge 154](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0154/){:target="_blank"}, I provided a couple of possible solutions in Pl/Perl, the widely available Perl integration within PostgreSQL.
<br/>
One task to solve, *Padovan numbers*, required to use *recursion*, and that is something not as simple as it could seem to implement using Pl/Perl.
<br/>
Why?
<br/>
Because Pl/Perl does not expose *Perl objects*, rather is a way to execute Perl within SQL objects (e.g., functions). What it means is that SQL objects are (clearly) the *first class objects* available, so you have always to use SQL functions to recurse.
<br/>
Except when you don't want to!
<br/>
<br/>
But let's start simple and see how to solve the problem.


## Padovan numbers

A *Padovan number* is a number defined as the sum of two preceeding numbers in the sequence. In particular:
- `P(0) = P(1) = P(2) = 1`, the first three elements of the sequence are equal;
- `P(n) = P(n - 3) + P(n - 2)`

This is great for recursion, because you can define a function in Pl/pgSQL as follows:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc154.padovan( i int )
RETURNS int
AS $CODE$
BEGIN
    IF i <= 2 THEN
       RETURN 1;
    END IF;

    RETURN pwc154.padovan( i - 3 ) + pwc154.padovan( i - 2 );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

## Translating to Pl/Perl and the problem of recursion

The above Pl/pgSQL function cannot be translated byte-by-byte to Pl/Perl; the following will not be possible:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
padovan_not_working( int )
RETURNS int
AS $CODE$
  return 1 if( $_[0] <= 2 );
  return padovan_not_working( $_[0] - 3 )
       + padovan_not_working( $_[0] - 2 );
$CODE$
LANGUAGE plperl;
```
<br/>
<br/>

In fact, the `padovan_not_working` is a function on the SQL side, and thus cannot be called by PlPerl as a Perl function.
<br/>
One, ease, solution, could be to accept the fact that the resulting function is an SQL object and interact with it accordingly:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc154.padovan_plperl( int )
RETURNS int
AS $CODE$
 return 1 if $_[0] <= 2;
 my ( $a, $b ) = ( $_[ 0 ] - 3, $_[ 0 ] - 2 );
 my $rs = spi_exec_query( "SELECT pwc154.padovan_plperl( $a ) + pwc154.padovan_plperl( $b ) AS p" );
 return $rs->{ rows }[ 0 ]->{ p };

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

As you can see, the function invokes itself by means of an SQL query.


## Using a closure

It is possible to use a closure to hold the reference to an anonymous code block, so that it is possible to implement the recursion as follows:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
plperl_padovan_recursive( int )
RETURNS int
AS $CODE$
  my $padovan;
  $padovan = sub {
    return 1 if $_[0] <= 2;
    return $padovan->( $_[0] - 3 ) + $padovan->( $_[0] - 2 );
  };

  return $padovan->( $_[0] );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

No need for queries, no need for external modules, **but** there are memory leaks due to the reference counting.



## Using `Sub::Recursive`

There is a module, named [`Sub::Recursive`](https://metacpan.org/pod/Sub::Recursive){:target="_blank"} that does exactly what I would like to go: allows to define an anonymous code block that can recursively invoke itself without any leak.
<br/>
The only drawback is that the function must be run as *Pl/Perl unsafe* because it needs to load a module outside of the PostgreSQL server (and of course, the module must be on the system, `cpanm` is your friend!):


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
plperl_padovan( int )
RETURNS int
AS $CODE$
use Sub::Recursive;
my $padovan = recursive {
    return 1 if $_[0] <= 2;
    return $REC->( $_[0] - 3 ) + $REC->( $_[0] - 2 );
};

  return &$padovan( $_[0] );
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

That's it! No need for queries, no need for `%_SHARED`, no need for closures (apparently), just Perl!
<br/>
**But, there is the need for `plperlu`!**
<br/>
The module provides the special *keyword* `recursive` that accepts a code reference with the closure `$REC` that holds a reference to the code block itself.



## Using `%_SHARED` ?

Another way to use recursion is by means of the Pl/Perl global hash `%_SHARED`, that is used to share whatever object you want across different functions. The idea is to share a function, so that it is possible to invoke it directly later on.
<br/>
The implementation could be as follows:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
plperl_padovan_init()
RETURNS VOID
AS $CODE$
  my $padovan;
  $padovan = sub {
    return 1 if $_[0] <= 2;
    return $padovan->( $_[0] - 3 ) + $padovan->( $_[0] - 2 );
  };

  $_SHARED{ padovan } = $padovan;
$CODE$
LANGUAGE plperl;


SELECT plperl_padovan_init();

CREATE OR REPLACE FUNCTION
plperl_padovan_shared( int )
RETURNS int
AS $CODE$
  my $padovan = $_SHARED{ padovan };
  return $padovan->( $_[0] );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The first function, `plperl_padovan_init`, installs a code reference `$padovan` into the global Pl/perl hash `%_SHARED`, so that other functions can obtain such code reference. The code is the same as in the other examples.
<br/>
Then the function is explicitly invoked, so that the code reference is installed.
<br/>
Later on, the `plperl_padovan_shared` function gets the code reference and use it as a normal function.




## Quick and dirt comparison

I've done a very short one-launch comparison among the approaches, excluding the one based on `%_SHARED` because it is very similar to the approach using the *pure* recursion via code reference. Just for the records, asking to the `%_SHARED` based approach to compute the `50`-th Padovan number requires around `0.5 secs` that is, as expected, in line with the other similar approaches.
<br/>
Increasing the Padovan number to compute makes the Perl approaches based on pure or `Sub::Recursive` really similar in terms of execution time. The approach that performs a query to use recursion is, as you can imagine, the slowest one and its performance decreases very quickly as the numbers grow.


The following table summarizes times depending on the generated number:

<br/><br/><br/>

{:class="table table-bordered"}
| Padovan number | `Sub::Recursive` | closure     | query       |
|----------------|------------------|-------------|-------------|
| 10             | 12 ms            | 20.39 ms    | 17.61 ms    |
| 20             | 0.51 ms          | 18.18 ms    | 12 ms       |
| 30             | 1.2 ms           | 17.9 ms     | 27.99 ms    |
| 40             | 31.9 ms          | 30.01 ms    | 323.4 ms    |
| 50             | 0.50 secs        | 0.52 secs   | 5.9 secs    |
| 60             | 12.76 secs       | 10.35 secs  | 99.77 secs  |
| 65             | 35.09 secs       | 35.78 secs  | 385 secs    |
| 70             | 142 secs         | 145 secs    | 1580 secs   |
| 71             | 187.89 secs      | 196.24 secs | 2122.8 secs |
|----------------|------------------|-------------|-------------|

<br/><br/>

It is not possible to keep increasing the Padovan number because of integer overflow, therefore I would have to adjust the functions to return `bigint`, but in any case I'm not expecting much different result trends.

# Conclusions

Recursion in Pl/Perl could be hard to implement and could require fancy approaches like the closure based ones.
First of all, you need to decide if you can deal with *untrusted* languages: if so, probably installing a module is the easiest and rightmost approach.
If you don't want to deal with untrusted code, you need to decide if you prefer to use a *pure* Perl approach, in such case a code reference is the choice, or you want to have something that can be invoked by other languages. The latter means using a more SQL-toward approach, while the former means sticking with a code refence, either used immediatly or by means of some sort of shared storage.
