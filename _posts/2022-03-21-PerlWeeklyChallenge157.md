---
layout: post
title:  "Perl Weekly Challenge 157: numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 157: numbers

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 157](https://perlweeklychallenge.org/blog/perl-weekly-challenge-157/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)



## COVID-19

Yes, I've *Covid-19* and I'm still at home, but I'm fine!



<a name="task1"></a>
## PWC 157 - Task 1

Really simple, just three lines of code:




<br/>
<br/>
```raku
sub MAIN( *@n where { @n.elems > 0 && @n.grep( * ~~ Int ) == @n.elems } ) {
    my $am = @n.sum / @n.elems;
    my $gm = ( [*] @n ) ** ( 1 / @n.elems.Rat );
    my $hm = @n.elems / @n.map( 1/* ).sum;

    "AM = $am GM = $gm HM = $hm".say;
}

 ```
<br/>
<br/>
The only *interesting* part is the `HM` computed by means of re-`map`ping the `@n` array into its reverse values.


<a name="task2"></a>
## PWC 157 - Task 2

This was about finding Brazilian numbers, those number that have one other base less than the number itself expressed by means of a single symbol. I used a `Bag` to store the count of the symbols, after having converted the number into another base, and then look for a single key.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 1 }, Bool :$verbose = True ) {
    for ( 2 .. $n - 2 ) {
        my $bag = $n.base( $_ ).comb.Bag;
        '1'.say and exit if $bag.keys.elems == 1;
    }

    '0'.say;
}


```
<br/>
<br/>

Therefore I convert the number into the current base by means of the `base` method, then I extract the array of symbols by means of `comb` and convert them into a `Bag`. Last, if the number of keys of the bag is one, then a single digit was used.

<a name="task1pg"></a>
## PWC 157 - Task 1 in PostgreSQL


A pure Perl implementation using three different functions:


<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc157;

CREATE OR REPLACE FUNCTION
pwc157.am( int[] )
RETURNS numeric
AS $CODE$
my $counter = 0;
my $sum     = 0;

for my $elem ( @{ $_[0] } ) {
    $counter++;
    $sum += $elem;
}

return $sum / $counter;
$CODE$
LANGUAGE plperl;



CREATE OR REPLACE FUNCTION
pwc157.gm( int[] )
RETURNS numeric
AS $CODE$
my $counter = 0;
my $mul     = 1;

for my $elem ( @{ $_[0] } ) {
    $counter++;
    $mul *= $elem;
}

return $mul ** ( 1 / $counter );
$CODE$
LANGUAGE plperl;


CREATE OR REPLACE FUNCTION
pwc157.hm( int[] )
RETURNS numeric
AS $CODE$
my $counter = 0;
my $sum     = 0;

my @n = map { 1 / $_ } @{ $_[0] };

for my $elem ( @n ) {
    $counter++;
    $sum += $elem;
}

return $counter / $sum;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

Nothing really fancy here: I loop over the array passed as argument, and that is an iterable object in PL/Perl, and count and sum/multiply/`map` the values.



<a name="task2pg"></a>
## PWC 157 - Task 2 in PostgreSQL

Here I used a couple of extenral Perl modules, so I need for `plperlu`. I used a base covnerter module, but since it does not allow for every numeric base, I had to check for exceptions. This means the PL/Perl implementation is not as accurate as the Raku counterpart. However, the idea is pretty similar:
- I convert the number and extract (by means of `split`) every symbol;
- I count for the symbols occurencies;
- if there is only one symbol, than the number looks good, otherwise it does not.


<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc157;


CREATE OR REPLACE FUNCTION
pwc157.brazilian( int )
RETURNS int
AS $CODE$
use Math::Base::Convert;
use Syntax::Keyword::Try;

for my $base ( 2 .. $_[0] - 1 ) {
    elog( DEBUG, "Base $base " );
    try {
        my $bc = Math::Base::Convert->new( 10, $base );
        my $n = $bc->cnv( $_[0] );
        elog( DEBUG, "Converted value $_[0] -> $n in base $base" );
        my @digits = split( //, $n );
        my %symbols;
        for my $d (@digits) {
            $symbols{ $d }++;
        }

        my $seen = 0;
        for my $k ( keys %symbols ) {
          $seen++ if $symbols{ $k } > 0;
        }

        return 1 if $seen == 1;

    } catch {
        elog( DEBUG, "Exception");
        next;
    }

}

return 0;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>
