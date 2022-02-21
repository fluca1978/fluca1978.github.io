---
layout: post
title:  "Perl Weekly Challenge 153: take it easy"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 153: take it easy

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 153](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0153/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>


<a name="task1"></a>
## PWC 153 - Task 1

This task was about producing *left factorial numbers*, where each value is computed by summing all the previously computed factorials.
<br/>
It took me more time to understand the implementation to do than the code to implement it!
<br/>


<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 0 } = 10 ) {
    my @factorials = lazy gather {
        for 0 .. $limit {
            take 1 if $_ <= 1;
            take [*] 1 .. $_ if $_ > 1;
        }
    };

    my @numbers = lazy gather {
        for 0 .. $limit {
            take 0 if $_ == 0;
            take 1 if $_ == 1;
            take @factorials[ 0 .. $_ -1 ].sum if $_ > 1;
        }
    };

    @numbers[ 0 .. $limit ].join( "\n" ).say;
}

 ```
<br/>
<br/>

I decided to implement the computation using a `lazy gather` approach. The `@factorials` array contains the factorials: for every index there is the factorial of such value. Then, `@numbers` computes the `sum` of all values previously stored into the array. Again, to be coherent, this is done using a `lazy gather` so that the implementation is somehow hyper-lazy!
<br/>
Note that `@numbers` is initialized with two defined elements, `0` and `1`. I don't agree with the fact that, begin `0!` equal to `1`, `!0` should not be zero itself.


<a name="task2"></a>
## PWC 153 - Task 2

A one liner: given an integer number, tell if the number is equal to the sum of the factorials of its digits.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {
    '1'.say and exit if $n.comb.map( { $_ <= 1 ?? 1 !! [*] 1 .. $_ } ).sum == $n;
    '0'.say;
}
```
<br/>
<br/>

I lied: it is actually two lines!
<br/>
I first split the `$n` into its digits by means of `comb`, then I `map` the resulting array with its factorial. In particular, I do compute the factorial by means of the reduction operator `[*]` only if the number is greater than `1`. Last, I do `sum` the result and cmpare it with the initial number `$n`.
