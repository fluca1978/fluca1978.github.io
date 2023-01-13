---
layout: post
title:  "Perl Weekly Challenge 108: Memory Layout and Bell Numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 108: Memory Layout and Bell Numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 108](https://perlweeklychallenge.org/blog/perl-weekly-challenge-108/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




<a name="task1"></a>
## PWC 108 - Task 1

The first task required to print out the memory location of a defined variable.
This is quite simple in both Raku and Perl, at least as the virtual machine provides information about the memory address.
<br/>
In Raku there is the `WHERE` method that can be invoked on every object to get the memory address value, **even if this is know to be unstable**: the memory location could change depending on the garbage collection.
<br/>
In any case, the resulting program is something like:

<br/>
<br/>
```raku
sub MAIN() {
    my Int $my-variable = 10;
    "%s @ %0xs (%d)".sprintf( $my-variable.^name, 
                              $my-variable.WHERE, 
                              $my-variable.WHERE )
                   .say;
}
```
<br/>
<br/>

That produces an output like the following:

<br/>
<br/>
```raku
% raku ch-1.p6  
Int @ 7f1b988ed2c8s (139756500341448)
```
<br/>
<br/>

There is much more boilerplate code to print out the values than computing them!


<a name="task2"></a>
## PWC 108 - Task 2

This task was about computing [Bell Numbers](https://en.wikipedia.org/wiki/Bell_number){:target="_blank"}. I decided to go straight to the implementation by means of *triangles*. The idea is to build up a triangle where each row is computed on values depending on the previous row. At the end, leftmost values of each row represent Bell Numbers.
<br/>
The program results in something like the following:

<br/>
<br/>
```raku
sub MAIN( Int :$base = 15 ) {

    # initialize the array used to compute Bell Numbers
    my @triangle = triangles( 1 );

    # compute all the triangle rows up to the base
    @triangle.push: triangles( $_, @triangle ) for 2 .. $base;

    # extract all the leftmost numbers of every row
    my @bell-numbers.push:  @triangle[ $_ ].head for 0 ..^ @triangle.elems;

    # all done, print out
    say "$base first Bell Numbers:";
    @bell-numbers.join( "\n" ).say;
}

```
<br/>
<br/>

The idea is to recursively compute every row of the `@triangle` by means of invoking the `triangles` function. Then I iterate on every row and extract the `.head` of the array stored on each row, that means getting the leftmost value, putting each of them in `@bell-numbers`. And last, I do print every number.
<br/>
I defined the `triangles` function as a `multi` sub, because there are two particular cases: `0` and `1` always return a single value. Therefore:

<br/>
<br/>
```raku
multi sub triangles( 0 ) { return 1; }
multi sub triangles( 1 ) { return 1; }
multi sub triangles( Int $base where { $base > 1 }, @triangle ) {
    # since $base starts from 2, the previous row is
    # 0, so $base - 2
    my @previous-row = | @triangle[ $base - 2 ];

    # add the leftmost element of this row
    my @current-row.push: @previous-row.tail;

    # iterate and compute the diagonal sum
    for 1 ..^ $base {
        @current-row.push: ( @previous-row[ $_ - 1 ] + @current-row[ $_ - 1 ] );
    }

    return @current-row;
}
```
<br/>
<br/>

As you can see, to compute a non-trivial row I extract the `@previous-row` from the `@triangle` (assuming every row is an array), and the leftmost value is the last (i.e., rightmost) value of the previous row.
Then I compute right values of the `@current-row` by summing values in diagonal with respect to the previous and current row. This is a simple straightforward implementation of the triangle.
