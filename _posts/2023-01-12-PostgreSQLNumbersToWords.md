---
layout: post
title:  "From Numbers to Words using Perl (and Lingua::)!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to convert a digit into a sentence with the power of Perl.

# From Numbers to Words using Perl (and Lingua::)!

A few days ago I came across some question on Facebook regarding the conversion of a number into its english representation. First of all, I hate Facebook with a passion and **I strongly encourage people that have questions related to PostgreSQL to use mailing lists and IRC channels**.
<br/>
Despite that, how can we convert a number, let's say `19` to `nineteen`?
<br/>
The first thing that came into my mind was the excellent **`Lingua::`** set of Perl modules. And since Perl is a very well supported language into every vanilla PostgreSQL instance, why not create a simple wrapper in PL/Perl?
<br/>
So, as trivial, as it is, [here you can find a simple implementation](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/num2words.pl){:target="_blank"} to translate a number into an english sentence.

## Installing `Lingua::EN::Numbers`

Before you can use my wrapper, you need to install the Perl module `Lingua::EN::Numbers` in your system, so that PostgreSQL can find it. One easy way to achieve this is by means of `cpanm`:

<br/>
<br/>
```shell
% cpanm Lingua::EN::Numbers
```
<br/>
<br/>

Once the module is installed, you can install the procedure.

## The `num2words` PL/Perl function: a first simple implementation

The function is a simple wrapper around the super powerful `num2en` function loadable via `Lingua::EN::Numbers`.
Since the PL/Perl function requires an external module, the function has to be defined as `plperlu`, therefore potentially unsafe.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
	num2words(  numeric default 0 )
RETURNS text
STRICT
AS $CODE$
   use Lingua::EN::Numbers qw/ num2en /;

   my ( $number ) = @_;

   num2en( $number );
$CODE$
LANGUAGE plperlu;
```
<br/>
<br/>

The function is defined as `STRICT`, and therefore it does return `NULL` on `NULL` input.
The function accepts a `number` input, therefore even a very large number, and passes it to `num2en` function, returning the result.
<br/>
As an example of invocations:

<br/>
<br/>
```sql
testdb=> select num2words();
 num2words
-----------
 zero
(1 row)
testdb=> select num2words( 19071978 );
                               num2words
------------------------------------------------------------------------
 nineteen million, seventy-one thousand, nine hundred and seventy-eight
(1 row)
testdb=> select num2words( 1.23 );
      num2words
---------------------
 one point two three
(1 row)
```
<br/>
<br/>


## The `num2words` function: a multi-language approach

It is possible to improve the function to support multiple languages, for example specifying the language as an argument.
One problem is that not all the `Lingua::*::Numbers` behave the same, so it is not simple to dynamically load a module and a function depending on the input argument. For example, in the English module the function to use is `num2word` while in the Italian language is `number_to_it`, so there is not a well established pattern name.
<br/>
Here there is a multilanguage implementation:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
	num2words( numeric default 0, text default 'en' )
RETURNS text
STRICT
AS $CODE$
   my ( $number, $language ) = @_;
   $language = 'en' unless( $language );

   if ( $language =~ /^en(glish)?$/i ) {
      use Lingua::EN::Numbers qw/ num2en /;
      num2en( $number );
   }
   elsif( $language =~ /^it(alian)?$/i ) {
     use Lingua::IT::Numbers qw/ number_to_it /;
     number_to_it( $number );
   }
   else {
   	elog( NOTICE, "Unsupported language $language" );
	return undef;
   }
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

The function accepts the locale (two letters) or the full language name (e.g., `italian`) in a case insensitive manner.
In every branch of the `if-elsif-else`  the appropriate module is loaded, and then the appropriated function is evaluated.
<br/>
Clearly, the function needs to access every language specific module, so before you can use the above PL/Perl function you need to install the Perl modules on the machine:


<br/>
<br/>
```shell
% sudo cpanm Lingua::EN::Numbers
% sudo cpanm Lingua::IT::Numbers
```
<br/>
<br/>



# Conclusions

I love the capability to melt Perl into PostgreSQL, giving the best of two worlds in a very quick and smart way!
