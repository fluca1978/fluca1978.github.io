---
layout: post
title:  "Perl Weekly Challenge 115: words in circles and numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 115: words in circles and numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 110](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0110/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 115 - Task 1
The first task was about discovering if  a list of words was *circular*, meaning if the first letter of a word was the final of another word and this is true for all the words.
<br/>
Initially this happened complex to me, then I realized that it does suffice to extract the list of all first-letters and compare it to the list of all last-letters: if the list are the same, the circle is possible.

<br/>
<br/>
```raku
sub MAIN( *@words  where { @words.elems > 0 } ) {

    # get a list of all initiali letters
    # and one of all tailing ones
    my @headings = @words.map: *.substr( 0, 1 );
    my @tailings = @words.map: *.substr( * - 1 );

    # if the sorted list are the same, there is a match!
    say @headings.sort ~~ @tailings.sort ?? 1 !! 0;

}

```
<br/>
<br/>


<a name="task2"></a>
## PWC 115 - Task 2

The second task was easier, at glance: given a list of numbers I need to find out the biggest multiple of `2` that can be done with the combination of the given digits. 
<br/>
I decided to put all the numbers into a list, checking if they are multiple of `2`, and then extract the `max` from the list.
<br/>
<br/>
```raku
sub MAIN( *@N where { @N.elems > 0 && @N.grep( * ~~ Int ).elems == @N.elems } ) {
    # create a list of integers
    # with only those that are divisible by 2
    my Int @numbers.push: .join.Int if .join.Int %% 2 for @N.permutations;

    # now ask for the max
    say @numbers.max;
}

```
<br/>
<br/>
