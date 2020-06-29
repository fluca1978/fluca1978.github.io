---
layout: post
title:  "Perl Weekly Challenge 67: integers and phone letters"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 64: integers and phone letters

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 64](https://perlweeklychallenge.org/blog/perl-weekly-challenge-064/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation here in Italy is still strange, at least to me.
We are not supposed to stay in office or work together, but can take trains without any limitations.
We don't know anything about the next school year, but children are able to attend summer camps.
<br/>
I'm currently working half at office, and half at home.


### Olivia
Olivia seems the cat she was before the car incident, except she does not jump at all.
So far, she never tried to get higher than the kitchen table. Before the incident she was always over the kitchen furnitures, on the bathroom mirror and on the wardrobe in our bedroom.


### Eyes

The last check up was terrible: the pressure has double despite the eye injection.
<br/>
I did undergo another injection, with more pain than the previous one, and I've increased the number of medications I need to take per day.
<br/>
The doctor is talking about another eye surgery to be done quickly, and of course, risky.
<br/>
I have to say I'm not concentrated at all, I'm really scared this time and I don't even know why I'm writing this down here.


<a name="task1"></a>
## PWC 67 - Task 1

The first task resulted to be more complex than it looked like. Given two integers, we have to produc a list of possible combinations of numbers in increasing order.

## First attempt (partially correct)

I decided to go as follows:
- declare a `@digits` array that contains the *alphabet* of digits I can use;
- start with the first digit from the alphabet and compose an array of `@combination` numbers up to the required `$n` size;
- since the digits in the `@combination` must be in increasing order, every cell of the array must be at least one step greater than the previous one;
- once the array `@combination` is complete, I can push it to the global list of found `@combinations`;
- I do iterate the process keeping fixed the first digit of the array and increasing all the remaining ones until the last digit of the array is the upper limit `$m`.

<br/>
Therefore, the code looks like:

```perl6
sub MAIN( Int :$m where { $m > 2 }  = 5,
          Int :$n where { $n < $m } = 2 ) {

    # available digits
    my @digits = 1 .. $m;
    # found combinations
    my @combinations;

    for @digits -> $start {

        # build the array of combinations starting with the
        # current digits, place another one that is increased by one
        # so to keep sorting...
        my @combination = $start;
        @combination.push: $start + 1;


        # ... and all an element until I've made the array
        while ( @combination.elems < $n ) {
            @combination.push( @combination[ *-1 ] + 1 );
        }


        # the last element of the array must be the value
        # I've got as parameter, otherwise iterate
        while ( @combination[ *-1 ] < $m ) {
            # clone the array because I'm going to change it!
            @combinations.push: Array.new( @combination );


            # increase by one every element, so it will be kept in
            # order
            for 1 ..^ @combination.elems {
                @combination[ $_ ] += 1;
            }
        }

        @combinations.push: @combination if ( @combination[ *-1 ] == $m );
    }

    @combinations.join( ", " ).say;
}
```

<br/>
Please note that, since I'm changing the content of the `@combination` array, I need to push a new array every time I completed an iteration, that is why I've `Array.new( @combintation)`.

The approach is partially correct because with an increasing number of `$n` it looses values.

## Second attempt

The second attemp was to generate the whole set of numbers made by `$n` digits and trim the result to only those that have an increasing sequence of digits:

```perl6
for ( 1 x $n ).Int ^..^ ( $m x $n  ).Int {
    my @digits = $_.comb;
    next if @digits.elems != $n;
    next if @digits.grep( * > $m );
    my $ok = True;
    $ok = False if ( @digits[ $_  ] >= @digits[ $_ + 1 ] ) for 0 ..^ @digits.elems - 1;
    @combinations.push: @digits if $ok;
}

```

I start from `1 x $n` which is to say `111` (`$n = 3`) and finish the loop to `555` (`$m=5`).
I extract all the digits into the `@digits` array, then check to have a correct length of digits (this can be omitted) and that no one digit is greater than the upper bound `$m` (done with `grep`).
Last, I check that every digits is not greater or equal of the subsequent one.
If all the above passes, the `@digits` array can be considered as a valid combination and is pushed into `@combinations` that is then printed.


## Third attempt

Based on the second attempt, I tried to remove the `if` with a `next` line. I [asked for help on IRC](https://colabti.org/irclogger/irclogger_log/raku?date=2020-06-29#l276){:target="_blank"} and so the loop becomes:

```perl6
    for ( 1 x $n ).Int ^..^ ( $m x $n  ).Int {
        my @digits = $_.comb;
        next if @digits.elems != $n;
        next if @digits.grep( * > $m );
        next if @digits.sort !~~ @digits;
        next if @digits.unique !~~ @digits;

        @combinations.push: @digits;
    }
```

*So the idea is that the array of `@digits` are good if they are made by exactly `$n` numbers, each one less or equal to `$m`, sorted and unique.*


<a name="task2"></a>
## PWC 67 - Task 2

Task 2 was a lot easier to me: produce a list of all possible letter combinations given a string that is placed on the phone keyboard.
<br/>
First of all I defined an hash `%letters` that, given a digit on the phone keyboard, provides an array of letters.
<br/>
Then I split the input string into single digits and pushed all the letter arrays into a *giant* array.
<br/>
Then producing the combination is a matter of passing the array to the `[X]` cross operator.
The final code is therefore:

```perl6
sub MAIN( Str $S ) {
    my %letters =
    1 => [ '_', ',', '@' ]
    , 2 => [ 'A', 'B', 'C' ]
    , 3 => [ 'D', 'E', 'F' ]
    , 4 => [ 'G', 'H', 'I' ]
    , 5 => [ 'J', 'K', 'L' ]
    , 6 => [ 'M', 'N', 'O' ]
    , 7 => [ 'P', 'Q', 'R', 'S' ]
    , 8 => [ 'T', 'U', 'V' ]
    , 9 => [ 'W', 'X', 'Y', 'Z' ];

    my @combinations;
    for $S.comb  -> $current {
        @combinations.push( %letters{ $current } ) if %letters{ $current }:exists;
    }

    ( [X] @combinations ).join( "\n" ).lc.say;
}
```

Most of the code is to create the `%letters` hash.

