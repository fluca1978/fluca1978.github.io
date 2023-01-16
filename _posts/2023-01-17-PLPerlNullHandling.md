---
layout: post
title:  "Handling NULLs and Empty values in PL/Perl"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to correctly detect an SQL NULL value in PL/Perl.

# Handling NULLs and Empty values in PL/Perl

Perl has a very simple **concept of truth**: *everything that is a non-empty, non-zero value is true!*
<br/>
It's that simple!
<br/>
<br/>
The problem with PL/Perl, the PostgreSQL internal language, is that SQL provides `NULL` values, that somehow are equivalent to Perl `undef` values. But unlike Perl, in SQL an empty string or a `0` value *is not `NULL`*!
<br/>
This implies that if you pass a `NULL` value to a PL/Perl function (or piece of code), Perl will evaluate them as `undef`, and this is good. The problem is that SQL values `0` and `''` (empty string) will be treated by Perl as *false* values while they are not.
<br/>
Luckily, the rule is simple: use Perl's `defined` operator to see if a value is `NULL` in the SQL sense.
<br/>

Let's see this with a [very trivial code example](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/plperl/catch_nulls.plperl){:target="_blank"}:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
plperl_catch_nulls( int, text )
RETURNS VOID
AS $CODE$
   my $arg = 1;

  for ( @_ ) {
      elog(INFO, "Input argument number $arg [$_] is false (as Perl)" ) if ! $_;
      elog(INFO, "Input argument number $arg [$_] is NULL (as SQL)" ) if ! defined $_;
      elog(INFO, "Input argument number $arg is valid [$_]" ) if defined( $_ ) && $_;
      $arg++;
  }


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Let's try the function with a few different set of arguments:

<br/>
<br/>
```sql
testdb=> select plperl_catch_nulls( 19, 'Hello World!' );
INFO:  Input argument number 1 is valid [19]
INFO:  Input argument number 1 is valid [Hello World!]
 plperl_catch_nulls
--------------------

(1 row)

testdb=> select plperl_catch_nulls( 0, '' );
INFO:  Input argument number 1 [0] is false (as Perl)
INFO:  Input argument number 1 [] is false (as Perl)
 plperl_catch_nulls
--------------------

(1 row)

testdb=> select plperl_catch_nulls( NULL, NULL );
INFO:  Input argument number 1 [] is false (as Perl)
INFO:  Input argument number 1 [] is NULL (as SQL)
INFO:  Input argument number 1 [] is false (as Perl)
INFO:  Input argument number 1 [] is NULL (as SQL)
 plperl_catch_nulls
--------------------

(1 row)

```
<br/>
<br/>

As you can see, when the argument is `NULL` the branches *false (as Perl)* and *NULL (as SQL)* are both triggered, while not `NULL` values are triggered only by the `defined` branch.

<br/>
If you like the ease of thinking of Perl, and do not want to go deep into the `NULL`/`defined` stuff, you can define your function as `STRICT` that makes PostgreSQL preventing the function invocation when at least one argument is `NULL`.
