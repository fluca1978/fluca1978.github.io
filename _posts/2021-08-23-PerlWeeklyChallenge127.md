---
layout: post
title:  "Perl Weekly Challenge 127: no need for coffee!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 127: no need for coffee!

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 127](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0127/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 127 - Task 1

The first task was about finding out if two *sets* of numbers have intersections. It was easy enough to be solved with a `.grep` of one set into the other:

<br/>
<br/>
```raku
sub MAIN() {
    # my @S1 = 1, 2, 5, 3, 4;
    # my @S2 = 4, 6, 7, 8, 9;
    my @S1 = 1, 3, 5, 7, 9;
    my @S2 = 0, 2, 4, 6, 8;

    "0".say and exit if @S1.grep: $_ for @S2;
    "1".say;
}
```
<br/>
<br/>


<a name="task2"></a>
## PWC 127 - Task 2

The second task was about recognizing if a given segment overlaps previous segments.
A segment is defined by the start value and the end value.
<br/>
First of all I created a function `is-conflict` that accepts two arrays and check if the two are in conflict, that means overlap: if any of them is empty or the value of the second is not within the range of the first, they do not overlaps.
<br/>
<br/>
```raku

sub is-conflict( @interval-a, @interval-b ) {
    return False if ! @interval-a || ! @interval-b;
    return False if @interval-a ~~ @interval-b;
    return True if @interval-a[ 0 ] <= $_ <= @interval-a[ 1 ] for @interval-b;
    return False;
}

```
<br/>
<br/>

The `MAIN` is therefore a matter of invoking thwe `is-conflict` function with every possible array slice for a given position. It is worth noting that I did something more in this task: I countered how many times a segment overlaps, and I did that by using a `%conflicts` hash that counts how many time the same segment is reported as in conflict. At the end, I print out a result.


<br/>
<br/>
```raku
sub MAIN() {
    my @intervals = [ [1,4], [3,5], [6,8], [12, 13], [3,20] ];

    my %conflicts;

    for 0 ..^ @intervals.elems -> $i {
        %conflicts{ @intervals[ $i ] }++ if is-conflict( $_, @intervals[ $i ] ) for @intervals[ 0 .. $i - 1 ];
    }

    for %conflicts.kv -> $what, $how-much {
        "$what conflicts $how-much times".say;
    }
}
<br/>
<br/>
The end result of this script is as follows:

<br/>
<br/>
```shell
% raku ch-2.p6
3 20 conflicts 2 times
3 5 conflicts 1 times
```
