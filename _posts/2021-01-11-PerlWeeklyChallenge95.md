---
layout: post
title:  "Perl Weekly Challenge 95: palindrome numbers and stacks"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 95: palindrome numbers and stacks

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 95](https://perlweeklychallenge.org/blog/perl-weekly-challenge-095/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



# My eyes...

It's hard.
<br/>
Period.
<br/>
<br/>
I have another medical examination this week, and I hope something better will arise.

<a name="task1"></a>
## PWC 95 - Task 1

The first task is very simple in Raku: find out if a number is palindrome.
<br/>
It does suffice to create a `MAIN` function that accepts only an integer number. After that, it is simply required to check the *stringified* version of the number is the same as its `flip`ped (i.e., reversed) string:

<br/>
<br/>
```raku
sub MAIN( Int :$N = 1 ) {
    say ~$N == ~$N.flip ?? '1' !! '0';
}
```
<br/>
<br/>


<a name="task2"></a>
## PWC 95 - Task 2

The second task was the implementation of a *Stack*, that can be easily achieved with the usage of arrays:

<br/>
<br/>
```raku

class SimpleStack {
    has Int @!elements;

    submethod BUILD {
        @!elements = Array.new;
    }

    method push( Int $n ) { @!elements.push: $n; }
    method pop()          {  @!elements[ @!elements.elems - 1 ]:delete; }
    method top()          { @!elements.reverse[ 0 ]; }
    method min()          { @!elements.min; }
    method print() { say $_ for @!elements.reverse;  }
}
```
<br/>
<br/>

In this way, every `push` appends to the end of the array the last value, and therefore the `top` is the first element of the reversed array, while the `min` is the overall mimimum value within the array.
The `pop` operation is implemented by means of deleting the last element in the array (that is the first element in the stack).
<br/>
Producing a `MAIN` to represent the task workflow is therefore really simple:

<br/>
<br/>
```shell
sub MAIN() {
    my SimpleStack $stack = SimpleStack.new;
    $stack.push(2);
    $stack.push(-1);
    $stack.push(0);
    $stack.pop;       # removes 0
    say $stack.top; # prints -1
    $stack.push(0);
    say $stack.min; # prints -1
}    
```
<br/>
<br/>
