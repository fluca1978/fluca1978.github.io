---
layout: post
title:  "Perl Weekly Challenge 133: quick and nested"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 133: quick and nested

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 133](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0133/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 133 - Task 1

The first task required to compute the first five *pandigital* numbers, where a pandigital number is a number that contains significative digits that correspond to the whole set of the digit alphabet.
<br/>
My idea was pretty simple:
- place all the digits I need to search for into the `@digits` array;
- start with a number that surely contains all the `@digits`, so `$start` is simply the concatenation of the `@digits`;
- loop from that `$start` to the infinity and see if the current number contains at least one digit of the `@digits` alphabet;
- if the current number contains at least one time every digit, produce it.

<br/>
I say "produce it" because I decided to implement the whole thing as a `lazy gather take` block of code, so the alghoritm does not have to compute all the required numbers at once.
<br/>
In the end, I do print the required number of elements, that is `$limit` with a default of five.
<br/>
So far, the code looks like the following:


<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 0 } = 5 ) {
    my @digits = 1 .. 9;
    @digits.push: 0;
    my $start =  @digits.join;

    my @pandigital = lazy gather {
        for $start ..^ Inf -> $current {
            next if $start ~~ / ^0+ /;
            my $found = 0;
            $found += $current.comb.grep( $_ ).so ?? 1 !! 0 for @digits;
            take $current if $found >= @digits.elems;
        }
    }

    @pandigital[ $_ ].say for 0 ..^ $limit;

}

```
<br/>
<br/>


<a name="task2"></a>
## PWC 133 - Task 2

This task was about producing a multiplication table, the one that you can find, or did find, in the back cover of the school notebooks.
<br/>
Quite frankly, the difficult was more in the printing step than in the calcolation phase. Since the task required also to find out unique numbers, I used two different arrays:
- `@table` contains the computations;
- `@distinct` contains unique numbers.
<br/>
The code looks like:

<br/>
<br/>
```raku
sub MAIN( Int $cols where { $cols > 0 } = 5, Int $rows where { $rows > 0 } = 3 ) {
    my @table;
    my @distinct;

    # table header
    "  x\t|\t".print;
    ( 1 .. $cols ).join( "\t" ).say;
    "--------|--------".print;
    ( "-" x 8 x $cols ).say;




    for 1 .. $rows -> $current-row {
        for 1 .. $cols -> $current-col {
            my $value = $current-row * $current-col;
            @table[ $current-row - 1 ].push: $value;
            @distinct.push: $value if ! @distinct.grep( $value );
        }
    }

    # print the table
    for 1 .. $rows {
        "  $_\t|\t".print;
        @table[ $_ - 1 ].join( "\t" ).say;
    }

    "\nDistinct values: ".say;
    @distinct.join( ', ' ).say;

}


```
<br/>
<br/>

I used the ugly `"\t"` tabular to quickly format the table when printing, but as you can imagine it would be better to build a more complex `printf` based approach.
<br/>
The result of the execution is like the following:

<br/<
<br/>
```shell
% raku ch-2.p6
  x     |       1       2       3       4       5
--------|------------------------------------------------
  1     |       1       2       3       4       5
  2     |       2       4       6       8       10
  3     |       3       6       9       12      15

Distinct values:
1, 2, 3, 4, 5, 6, 8, 10, 9, 12, 15

```
