---
layout: post
title:  "Perl Weekly Challenge 119: simpler than it seems"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 119: numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 119](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0119/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 119 - Task 1

The first task was about *rotating nibbles*, that means convert an integer to a bit string, and switch the half-octect, printing out the resulting number. One constraint that makes it really simple is that the incoming number must be less than `256`, that means we have a single octet and thus two nibbles.

<br/>
<br/>
```raku
sub MAIN( Int $N where { $N < 256 && $N >= 0 } ) {

    # create an 8 digits binary string
    '%08d'.sprintf( $N.base( 2 ) )
    # separate each digit into an array
    .split( '', :skip-empty )
    # rotate by four elements
    .rotate( 4 )
    # recombine
    .join
    # reparse as binary
    .Str
    .parse-base( 2 )
    # and print
    .say;

}

```
<br/>
<br/>
A single line does it all:
- I convert the `SN` to binary by means of `base( 2 )`;
- I print it as an eight digit string, placing leading zeroes by means of `printf`;
- I split the string into a single array of digits;
- I use `rotate` that takes `4` elements of the array and rotate to the left the elements;
- I `join` the obtained digits and convert them as a string;
- I compute the base 10 value by means of `parse-base` and then I print out the result.


<a name="task2"></a>
## PWC 119 - Task 2

The second task was about generating a *strange* sequence of numbers, and printing out only the selected one.
I used the `gather` and `take` construct here to load a lazy array of numbers:

<br/>
<br/>
```raku
sub MAIN( Int $N where { $N > 0 } ) {

    my @numbers = lazy gather {
        for 1 .. Inf {
            next if $_ ~~ / (11)+ /;
            next if $_ ~~ / <[04..9]> /;

            take $_;

        }
    }

    @numbers[ $N - 1 ].say;
}

```
<br/>
<br/>

I skip, by means of `next` the values that do not correspond to the requirements, those with repetitions of `1` and digits that are not `2`, `3` or `1` itself.
In the last, I print the required number, even if it took me a while to figure out that the exercise was asking about a numbering starting from one and not from zero as per a regular array.

## Wait a minute: what about PWC 118?

I was on holiday, without any computer at all!
