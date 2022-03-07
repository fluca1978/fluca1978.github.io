---
layout: post
title:  "Perl Weekly Challenge 155: Fortune and Pisano in Primes"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 155: Fortune and Pisano in Primes

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 155](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0155/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for other PWC in the past, I've done also a couple of possible implementations in PostgreSQL:
<br/>
- [Task 1 in PostgreSQL](#task1pg)
- [Task 2 in PostgreSQL](#task2pg)

<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)

<a name="task1"></a>
## PWC 155 - Task 1

Not so hard: compute the *Fortunate* numbers, those numbers that sum with a `pn` give a prime number. Here, `pn` stands for the multiplication of all `n` prime numbers. The task added to find out all first *8* unique Fortunate numbers.
<br/>



<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 0 } = 8 ) {
    my @fortunate-numbers = lazy gather {
        for 2 .. Inf {
            my @pn = ( 1 .. $_ ).grep: *.is-prime;
            next if ! @pn;
            for @pn.max + 1 .. Inf -> $m {
                take $m and last if ( ( [*] @pn ) + $m ).is-prime;
            }
        }
    }

    my @unique-fortunate-numbers;
    my $last-number = 0;
    while ( @unique-fortunate-numbers.elems < $limit ) {
        my $fortunate = @fortunate-numbers[ $last-number++ ];
        @unique-fortunate-numbers.push: $fortunate if ! @unique-fortunate-numbers.grep: * ~~ $fortunate;
    }

    @unique-fortunate-numbers[ 0 .. $limit ].sort.join( "\n" ).say;
}


 ```
<br/>
<br/>

The `@fortunate-numbers` is a lazy array computed by means of `@pn`, which is the list of all prime numbers up to a given number, and then a new loop is done to find out the smallest `$m` number greater than any value in `@pn` that, when summed with the multiplication of all values in `@pn` gives a prime.
<br/>
To find out only unique numbers, I loop again to build a `@unique-fortunate-numbers` array and stop the searching for as soon as the elements in the array are at the limit.


<a name="task2"></a>
## PWC 155 - Task 2

*Pisano Period*: finding out the repetition of the sequence of Fibonacci when every element is *modulo* a given number.

<br/>
<br/>
```raku
sub MAIN( Int $nth where { $nth >= 3 } = 3,
          Int $accuracy where { $accuracy > 1 } = 5,
          Bool :$verbose = False ) {

    my @fibonacci = 0, 1, 1, 2, * + * ... *;
    my @pisano    = @fibonacci.map: * % 3;

    # with nth >= 3 the period is always even
    my $period = 2;

    # build $accuracy arrays to check
    my @checking.push: @pisano[ ( 0 + $period * $_ ) .. ( $period * ( $_  + 1 ) ) - 1 ] for 1 .. $accuracy;

    # while the array are not all the same, grow them and recheck
    while ( not [eqv] @checking ) {
        $period += 2;
        @checking = ();
        @checking.push: @pisano[ ( 0 + $period * $_ ) .. ( $period * ( $_  + 1 ) ) - 1 ] for 1 .. $accuracy;
    }

    @checking.join( "\n" ).join( ',' ).say if $verbose;
    "Pisano period $nth is $period".say;

}

```
<br/>
<br/>

The idea is to build a lazy sequence `@pisano` that `map`s every element of the Fibonacci's serie modulo `3` as asked.
Since, when the modulo applied is greater or equal to 3, the period (i.e., the size of repeating part of the sequence) is always even, I start with a `$period` of `2`. Then I extract a number of arrays of size `$period` equal in number to `$accuracy`, e.g., 5 arrays of size 2.
Last, I check if the extracted array are all the same, i.e., are `eqv`. If they are, the `$period` is found, otherwise I increase the period of a even value and do it again.


<a name="task1pg"></a>
## PWC 155 - Task 1 in PostgreSQL

Pure Pl/Perl implementation, with a single function (that has nested anonymous `sub`s).


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc155.fortunate( int )
RETURNS SETOF integer
AS $CODE$

# a subroutine to see if a number
# is prime
my $is_prime = sub {
   return 1 if $_[0] == 1;
   return 1 if $_[0] == 2;
   for my $i ( 2 .. $_[0] - 1 ) {
       return 0 if $_[0] % $i == 0;
   }

   return 1;
};

# generates the first n primes
my $generate_primes = sub {
   my @primes;
   for my $p ( 2 .. 99999 ) {
       push @primes, $p if $is_prime->( $p );
       return @primes if @primes == $_[0];
   }
};

my $max = sub {
   my $max = 0;
   for (@_) {
       $max = $_ if $_ > $max;
   }

   elog( DEBUG, "MAX = $max  in " . join( ',', @_ ) );
   return $max;
};


my $pn = sub {
   my $result = 1;
   for ( @_ ) {
       $result *= $_;
   }

   return $result;
};

my $limit = $_[0] || 8;
my %unique;

for my $n ( 1 .. 999999 ) {
    # generate the first n primes
    my @primes = $generate_primes->( $n );
    my $start  = $max->( @primes ) + 1;
    elog( DEBUG, "Primes = " . join( ',', @primes ) . " with max = $start" );
    for my $m ( $start .. 999999 ) {
         my $fortunate = $pn->( @primes ) + $m;
         elog( DEBUG, "Computing $m -> " . $pn->( @primes ) . " + $m = $fortunate = " . $is_prime->( $fortunate ) );
         next if ! $is_prime->( $fortunate );
         $unique{ $m }++;
         next if $unique{ $m } > 1;
         return_next( $m );
         last;
    }


    $limit--;
    last if $limit <= 0;

}



return undef;
$CODE$
LANGUAGE plperl;


```
<br/>
<br/>

The function accepts a single argument, that is the number of Fortunate numbers to generate as *uniques*.
There are a couple of inner subs like `$max` (tom compute the max of an array), `$is_prime` (to see if a given number is prime and `$generate_primes` to generate the first `n` primes.
<br/>
Last `$pn` computes the multiplication of a given array.
<br/>
With that in place, I do loop from `1` to almost a big number (to simulate `Inf`), compute the primes up to such number, compute the max within such set of primes, and then loop again to search for another number `$m` that summed to the multiplication of ptrimes gives another prime.
If I find, I do `return_next` to append such value to the result set and exit the innermost loop, that is I start over computing a larger set of primes.
<br/>
Once the list of numbers is of the right size, I exit also the outer loop and terminate the function.



<a name="task2pg"></a>
## PWC 155 - Task 2 in PostgreSQL

This time I snooped for other people solutions, and found something really interesting in [Abigail's](https://github.com/manwar/perlweeklychallenge-club/pull/5751/commits/8c4b7ad7bd474c48af5bebdb566412d30101f28e){:target="_blank"}: the period can be computed depending on where the required size if found in the Fibonacci's serie.
<br/>
Therefore the function results in a very simple implementation:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc155.pisano_period( int )
RETURNS integer
AS $CODE$

my @fibonacci;
for ( 1 .. 999999 ) {
    push @fibonacci, 1 if $_ <= 1;
    push @fibonacci, @fibonacci[ -1 ] + @fibonacci[ -2 ];
    elog( DEBUG, "Fibonacci is " . join( ',', @fibonacci ) );
    last if @fibonacci[ -1 ] == $_[0];
}

# get the index
my $index = $#fibonacci + 1;
elog( DEBUG, "$_[0] found on index $index");
return $index * 2 if $_[0] >= 3 and $index % 2 == 0;
return $index * 4 if $_[0] >= 5 and $index % 2 == 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The idea is to generate an array of `@fibonacci` stopping as soon as the required input argument is found.
Then I compute the 1-based `$index` and see if it is even or odd and if the starting period dimension was greater or equal than 3 or 5, and all it's done!

<br/>
<br/>

``` sql
testdb=> select pwc155.pisano_period( 3 );
DEBUG:  Fibonacci is 1,1
DEBUG:  Fibonacci is 1,1,2
DEBUG:  Fibonacci is 1,1,2,3
DEBUG:  3 found on index 4
pisano_period
---------------
8

```
