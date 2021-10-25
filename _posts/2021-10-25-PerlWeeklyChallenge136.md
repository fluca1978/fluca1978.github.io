---
layout: post
title:  "Perl Weekly Challenge 136: not too hard"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 136: not too hard

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 136](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0136/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)



## Linux Day 2021

I gave a brief and quick presentation about Raku and how I feel about it. The talk is in italian, and can be seen as an extracted and cut video on [Odysee](https://odysee.com/@fluca1978:d/2021_LINUXDAY_RAKU:7){:target="_blank"}.


<br/>
<br/>
<center>
<iframe id="lbry-iframe" width="560" height="315" src="https://odysee.com/$/embed/2021_LINUXDAY_RAKU/71d1f405dc6bea384c040da06c0285c17229f6de?r=H6VMSo61DL7wbefVnV8hMQRoNY7ANu89" allowfullscreen></iframe>
</center>
<br/>
<br/>






<a name="task1"></a>
## PWC 136 - Task 1

The first task was a one-liner for me: find out if, given two numbers, their Greatest Common Divisor is a power of `2`.

<br/>
<br/>
```raku
sub MAIN( Int $m where { $m > 0 }, Int $n where { $n > 0 } ) {
    ( [gcd] $m, $n ) %% 2 ?? '1'.say !! '0'.say;
}

```
<br/>
<br/>

It work as follows:
- it computes the Greatest Common Divisor by using the `gcd` built-in operator;
- is compares the result with the `%%` operator to see if it is a multiple of `2`, that means also if it is a positive power of `2`;
- the ternary operatory prints out `1` or `0` depending respectively on success or failure.



<a name="task2"></a>
## PWC 136 - Task 2

The second task involved somehow the Fibonacci's serie and its sum: given a positive number, find out all available combinations of a trimmed Fibonacci's series in order to match exactly the given sum.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 1 } ) {
    my @fibonacci;
    @fibonacci.push: 1, 1;
    my %solutions;

    # compute a reduced fibonacci sequence up to the sum of the number given
    @fibonacci.push: @fibonacci[ * - 1 ] + @fibonacci[ * - 2 ]  while $n > ( @fibonacci[ * - 1] + @fibonacci[ * - 2 ]);


    # iterate over all the available numbers, and compute the sum
    # and if the sum does match, add the array to the hash of solutions
    # with a stringified key representation
    %solutions{ $_.join( ' + ') } = $_ if ( ( [+] $_ ) == $n ) for @fibonacci.combinations.unique;

    # print the number of keys
    say %solutions.keys.elems;
    # and print all the sums
    .join( " + " ).say for %solutions.values;

}


```
<br/>
<br/>

First of all, I compute a reduced `@fibonacci` array of numbers: numbers are pushed to the array only if their Fibonacci sum clause does not result greater than the given number.
<br/>
Then I build an hash `%solutions` indexed with a string representation of the current Fibonacci's sequence of numbers, and with a value of a subset of the Fibonacci's array if the sum of such array is the searched for value.
<br/>
Using an hash guarantees that I'm not going to find out duplicated sequences, and therefore it does suffice to print out the number of keys and, optionally, the human readable sums.
<br/>
As an example of output:

<br/>
<br/>

```shell
 % raku ch-2.p6 16
4
3 + 5 + 8
1 + 2 + 5 + 8
1 + 2 + 13
3 + 13

```
<br/>
<br/>
In the above there are `4` solutions, that are displayed as expanded sums in the subsequent lines.
```

```
