---
layout: post
title:  "Perl Weekly Challenge 122: mangling numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 122: mangling numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 122](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0122/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 122 - Task 1

The first task was about computing the *average* of a stream of numbers. Since it is not clear to me if the word *stream* here means a continuos input from the user, I provided two different `MAIN`s: one that gets an array of numbers, and one that reads from `STDIN`.
<br/>
The application is as follows:

<br/>
<br/>
```raku
multi sub MAIN( *@N where { @N.elems == @N.grep( * ~~ Int ).elems }) {
    my @average;
    @average.push: (@N[ 0 .. @average.elems - 1 ].sum + $_) / ( @average.elems + 1 ) for @N;


    @average.say;
}


multi sub MAIN() {
    "Insert a number at time".say;

    my @average;
    my $sum = 0;
    for lines() {
        "Not a number".say && next if $_ !~~ / \d+ /;
        $sum += $_;
        @average.push: $sum / ( @average.elems + 1 );
        "Average trend so far: { @average.say }";

    }
}
 ```
<br/>
<br/>

The *streaming* `MAIN` (the second one) is a little boring: it accumulates the sum of provided numbers into a `$sum` variable, and pushes the computed average into an `@average` array. I don't keep track of how many numbers have been provided, since `@average` contains that information too. Also, I do test for *good numbers* before computing any value.
<br/>
The first `MAIN` does pretty much the same, but the `@average` array is populated within a `for` loop that scans every input number, and the average value is made by summing the current value with the *slice* of previous values on the input `@N` array.


<a name="task2"></a>
## PWC 122 - Task 2

The second task was about finding out all possible sequences of available *baseball scores* that provide a final score specified as input.
<br/>
I decided to let Raku do a lot of computations, producing all available permutations of numbers and throwing away those that do not provide the right final score. I could not come up with a simpler solution, from the point of view of code, than this, but clearly it is strongly de-optimized!
<br/>

<br/>
```raku
sub MAIN( Int $S ) {
    my @available-points = 1, 2, 3;
    my $begin = ( @available-points[ 0 ] x $S ).Int;
    my $end   = ( @available-points[ * - 1 ] x $S ).Int;
    my @scores;


        for ( $begin .. $end ) {
            my @digits = $_.split( '', :skip-empty ).grep( * == any( @available-points ) );
            next if ! @digits.grep: $_ for @available-points;
            next if @digits.sum != $S;
            my $score =  @digits.grep( * == any( @available-points ) ).join;;
            @scores.push: $score if ( ! @scores.grep( $score ) );

        }


    @scores.join( "\n" ).say;
}
```
<br/>
<br/>

The idea is that, assuming the final score `$S` as `4`, I got all values between `1111` and `3333`. After that I check if the number, split into its `@digits` can give me a sum up to `$S`, and in the case I place it into the `@scores` array, after trimming out all zeros. If the number includes digits that are not within the `@available-points` or if the sum of those digits is not the one I'm looki9ng for, I discard the number.
<br/>
Trimming out the `0` from a number means that the `@scores` array is going to include duplicates (e.g., `1002` and `1200` will both result in `12`), so at the end I apply `unique` to get a single value out of all duplicates.
