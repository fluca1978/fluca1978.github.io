---
layout: post
title:  "Perl Weekly Challenge 186: zip and unicode"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 186: zip and unicode

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 186](https://perlweeklychallenge.org/blog/perl-weekly-challenge-186/){:target="_blank"}.

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
## PWC 186 - Task 1

Given two lists, of identical size, *zip* them.
<br/>
Raku has the `Z` zip operator that does just that, so my first attemp was:

<br/>
<br/>
```raku
sub MAIN() {
    my @a = < 1 2 3 4 >;
    my @b = < a b c d >;

    say |(|@a Z |@b);
}
 ```
<br/>
<br/>

One *problem* is that all the list have to be flattened by means of the pipe, this to ensure a single list is returned.
One could also use some hyper operator, like the comma, with something like `|( |@a >>,<< |@b)`, but as you can see, the flattening is still needed.


<a name="task2"></a>
## PWC 186 - Task 2
Make an UTF-8 based string printable: luckily every string in Raku has a method `smaemark` that provides a transformation from an UTF-8 char to an ascii one.

<br/>
<br/>
```raku
sub MAIN( Str $input ) {
    $input.samemark( 'aeiou' ).say;
}


```
<br/>
<br/>



<a name="task1plperl"></a>
## PWC 186 - Task 1 in PostgreSQL PL/Perl

Here I need to implement a `zip` (anonymous) function that iterates over all elements of the two arrays provided as input and push one element at the time into a third, resulting, array.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc186.task1_plperl( text[], text[] )
RETURNS text[]
AS $CODE$
my ( $left, $right ) = @_;

my $zip = sub {
   my ( $left, $right ) = @_;
   my $zipped = [];

   while ( $left->@* ) {
         push $zipped->@*, shift $left->@*, shift $right->@*;
   }

   return $zipped;
  };

  return $zip->( $left, $right );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I use the `shift` command to get a value out of each array at the same time, so I don't need to keep an index.


<a name="task2plperl"></a>
## PWC 186 - Task 2 in PostgreSQL PL/Perl

Here I need to use a module, like `Unicode::Normalize` that can provide me a way to transform a string. In partricular, the `NFD` method splits the unicode characters in a base and an extension, so that I can keep in the result only *simple* chars.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc186.task2_plperl( text )
RETURNS text
AS $CODE$

use Unicode::Normalize;

my ( $input ) = @_;
my @chars;

my $nfd = NFD $input;
for ( $nfd =~ / (\X) /xg ) {
    $_ =~ / (.) /x;
    push @chars, $1;
}

return join('', @chars );
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

I first get an `NFD` string, than I globally match against every expanded character (`\X`), and for every character I got only the first part. Such part is captured into `$1` and pushed into the `@chars` array, that is then joined and returned to the caller.


<a name="task1plpgsql"></a>
## PWC 186 - Task 1 in PostgreSQL PL/PgSQL

Using the `FOREACH` loop approach and the `array_append` function it is quite simple (but verbose) to implement the zipping:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc186.task1_plpgsql( l text[], r text[] )
RETURNS text[]
AS $CODE$
DECLARE
        res text[];

BEGIN
    FOR i IN 1 .. array_length( l, 1 ) LOOP
        res := array_append( res, l[ i ] );
        res := array_append( res, r[ i ] );
    END LOOP;

    RETURN res;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>




<a name="task2plpgsql"></a>
## PWC 186 - Task 2 in PostgreSQL PL/PgSQL

Here I trickly just call the Perl implementation to handle unicode chars!

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc186.task2_plpgsql( t text )
RETURNS text
AS $CODE$
BEGIN
        RETURN pwc186.task2_plperl( t );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
