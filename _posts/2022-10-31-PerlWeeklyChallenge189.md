---
layout: post
title:  "Perl Weekly Challenge 189: I have no time this week (again)!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 189: I have no time this week (again)!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 189](https://perlweeklychallenge.org/blog/perl-weekly-challenge-189/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 189 - Task 1

You are given a character and a list of characters, and you must to find out the first lexicographically subsequent char in the list of the former one.

<br/>
<br/>
```raku
sub MAIN( Str $k, *@letters ) {
    @letters.grep( $k le * ).sort.head.say;
}

 ```
<br/>
<br/>

This is a one liner: I extract a sublist of all the chars that are greater than the given char `$k`, than I `sort` and keep the `head` (first value).


<a name="task2"></a>
## PWC 189 - Task 2

Given an array of integers, find out the smallest slice with the degree of the array.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems
                                                                && ! @numbers.grep( *.Int < 0 ) } ) {

    my $degree = Bag.new( @numbers ).values.max;

    for 2 .. @numbers.elems {
        my @list = @numbers.rotor( $_ => -1 * ( $_ -1 ) );
        for @list -> $current {
            $current.say and exit if Bag.new( $current.Array ).values.max == $degree;
        }
    }
}

```
<br/>
<br/>

First of all, I compute the `$degree` of the input array by means of finding out the `max` within the `values` of the `Bag`, that counts the frequency for me.
<br/>
Then I compute a `rotor` of the size of `2` up to the size of the whole array. For any sublist from the `rotor`, I compute again the degree by means of the `Bag` and in the case this is the first array that matches the degree, I print it out and exit.
The trick here is that the `rotor` is called with the less common *pair syntax* that takes the key as the size of the rotor and the negative value as how many values can overlap, that is as the size increases the function produces more and more overlapping arrays.
