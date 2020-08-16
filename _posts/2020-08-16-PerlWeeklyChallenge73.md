---
layout: post
title:  "Perl Weekly Challenge 73: reduction to min"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 73: reduction to min

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 73](https://perlweeklychallenge.org/blog/perl-weekly-challenge-073/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### Sunday Rush

I was on vacation this week, so I had to complete this PWC on sunday as I got home. The fun part is that tomorrow another PWC will hit my inbox.
<br/>
Luckily, this PWC was not too difficult to me.

<a name="task1"></a>
## PWC 73 - Task 1

The first task was related to find out the minimum in an array window. Since Raku allows for array slices, and the `[min]` reduction operator does the trick, the only difficult in this task was to correctly compute the size of the window to work on:


<br/>
<br/>
```raku
sub MAIN( Int $S where { 0 < $S },
          *@A where { @A.elems > $S } ) {

    my @mins;
    for 0 .. ( @A.elems - $S ) {
        @mins.push: [min] @A[ $_ ..^ ( $_ + $S ) ];
    }

    @mins.join( ',' ).say;
}

```
<br/>
<br/>

I decided to put `$S` as first positional to allow a slurpy parameter to consume all remaining numbers. 
<br/>
After that I iterated over the windows, computing the minimum value and assigning it to the `@mins` array.

<a name="task2"></a>
## PWC 73 - Task 2


The second task required to find out the minimum element on the left side of an array element of integers. Again, the `[min]` reduction operator was the best candidate to me, with the only exception that in case there was not a value less than the current one the special value of zero has to be used.


<br/>
<br/>
```raku
sub MAIN( *@A where { @A.grep( * ~~ Int ).elems == @A.elems } ) {
    my @mins;
    for 0 ..^ @A.elems {
        my $current-min = $_ == 0 ?? 0 !! [min] @A[ 0 ..^ $_ ];
        @mins.push: $current-min < @A[ $_ ] ?? $current-min !! 0;
    }

    @mins.join( ',' ).say;
}

```
<br/>
<br/>

Since slurpy arrays cannot be typed, I had to ensure that every element of the array was an integer.
<br/>
The first element of the array cannot have nothing on its left, so it defaults to zero. The other elements assume the `[min]` of the slice from the first element to the current one. If such value, stored into `$current-min` is less than the current element `@A[$_]` then it is pushed to the array of `@mins`, otherwise zero is pused.
