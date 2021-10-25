---
layout: post
title:  "Perl Weekly Challenge 131: no coffee, no elegance"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 131: no coffee, no elegance

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 131](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0131/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 131 - Task 1

The first task was about splitting an incoming set of ordered integers into array, so that each array contains consecutive values.
<br/>
I decided to use an array `@results` that acts as *an array of arrays* where each embedded array is one of the resulting arrays required by the task. It is quite clear that the first array in `@results` is the first element in the list of input integers.
<br/>
Then it does suffice to iterate over the array of integers (skipping the first element) and see if *the last element of the last sub-array* has a distance of `1` from the current value. If the distance is `1` the value is pushed into the very last array in the `@results`, otherwise it means a new sub array must be created with the starting element as the current one.
<br/>
The code therefore looks like:

<br/>
<br/>
```raku
sub MAIN( *@values where { @values.elems > 1 && @values.grep( * ~~ Int ).elems == @values.elems } ) {
    my @results;
    @results.push: [ @values[ 0 ] ];
    for 1 ..^ @values.elems {
        if @results[ * - 1 ][ * - 1 ] - @values[ $_ ] == -1 {
            @results[ * - 1 ].push: @values[ $_ ];
        }
        else {
            @results.push: [ @values[ $_ ] ];
        }
    }

    @results.say;
}
```
<br/>
<br/>

The only trick, thus, is to use the *last to last element* as `[ * - 1 ][ * - 1 ]`.



<a name="task2"></a>
## PWC 131 - Task 2

The second task was about recognizing how many *starting delimiters* and *closing delimiters*, passed as input, are there in another input string.
<br/>
Assuming therefore that the `$delimiters` input string is made by pairs, it is quite easy to iterate over the pairs and match the `$needle` searching for string. The results are pushed into two different arrays, that are then printed on separated lines:


<br/>
<br/>
```raku
sub MAIN( Str $delimiters where { $delimiters.chars %% 2 },
          Str $needle where { $needle.chars >= 2 } ) {
    my Str @openings;
    my Str @closings;

    for $delimiters.split( '', :skip-empty ) -> $open, $close {
        @openings.push: $open if $needle ~~ / $open /;
        @closings.push: $close if $needle ~~ / $close /;
    }

    @openings.join.say;
    @closings.join.say;

}

```
<br/>
<br/>
