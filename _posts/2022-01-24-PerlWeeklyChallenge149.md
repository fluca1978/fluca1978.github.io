---
layout: post
title:  "Perl Weekly Challenge 149: Fibonacci & squares"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 149: Fibonacci and squares

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 149](https://perlweeklychallenge.org/blog/perl-weekly-challenge-149/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg) and a [CTE only solution](#task2pgb)





<a name="task1"></a>
## PWC 149 - Task 1

The first task requires to find out first numbers those digits, summed, provide a number in the Fibonacci's series.
My implementation is as follows:


<br/>
<br/>
```raku
sub MAIN( Int $N where { $N > 0 } ) {
    my @fibonacci = 1, 1, * + * ... *;
    my $fibonacci-index = $N;
    my @numbers = lazy gather {
        for 0 .. Inf -> $i {
            take $i and $fibonacci-index += $N if @fibonacci[ 0 .. $fibonacci-index ].grep( $i.split( '' ).sum );;
        }
    };

    @numbers[ 0 .. $N - 1 ].join( ',' ).say;
}
 ```
<br/>
<br/>

I define a `@fibonacci` lazy sequence, and keep an index of what part of the series I want to inspect. In fact, being a lazy list, a normal `grep` will not work as expected, so I slice the list in partitions.
<br/>
Then I define the `@numbers` of the solution as a lazy list, where I `take` a number only if the sum of its digits is within the '@fibonacci` slice.
<br/>
In the end, I do print the slice of the `@numbers` so making such values computed and materialized.


<a name="task2"></a>
## PWC 149 - Task 2

The second task was much more complicated to me to understand, and it is probably for this reason that I came up with a very inefficient solution: finding out, for a given numeric base, the largest number that is a square and has non-repeating digits.

<br/>
<br/>
```raku
sub MAIN( Int $base where { $base >= 2 } ) {
    my @alphabet = 0 .. min( $base, 9 ) - 1;
    @alphabet.push: |('A' .. 'Z' )[ 0 .. $base - 10 - 1 ] if $base > 10;

    my $solution = 0;
    for @alphabet.permutations {
        my $current = $_.join.parse-base( $base );
        $solution = $current if $current > $solution && $current.sqrt ~~ $current.sqrt.Int;
    }


    $solution.base( $base ).say;
}

```
<br/>
<br/>

Given the `$base`, I produce the `@alphabet` with all the digits and letters if required.
Then I compute all the permutations of the alphabet, and compute the `$current` value by parsing the base.
Producing the alphabet guarantees me that every permutation is unique, that means it does not have repeating digits.
If the computed value is greater than the previously computed and has a square root that is an integer, than it is good to keep as a possible solution.
At the end, the greatest solution is printed.
