---
layout: post
title:  "Perl Weekly Challenge 130: quick"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 130: quick

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 130](https://perlweeklychallenge.org/blog/perl-weekly-challenge-130/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 130 - Task 1

The first task was quite simple: it required to find out a number in a list of arguments that appears an odd number of times. It is simple enough to solve it by means of `grep` and `for`:

<br/>
<br/>
```raku
sub MAIN( *@values where { @values.elems > 1 && @values.grep( * ~~ Int ).elems == @values.elems  } ) {
    $_.say and exit if  @values.grep( $_ ).elems !%% 2 for @values;
}

```
<br/>
<br/>
It must be read from right to left: the `for` loop values `$_` to every single element of the array, then the `if` tests if the `$_` is found in the array, and `elems` counts how many time it appears. If the number of times is odd, that is *is not even* (`!%% 2`), the value can be printed out. This is do with a `say`, and then the program exits with a low precedence `and exit`, because the task states that only one number can appear an odd even of times.



<a name="task2"></a>
## PWC 130 - Task 2

The second task was about verifying if a tree is a balanced binary tree. *I hate trees with a passion!*
<br/>
I implemented a class `Node` that has the current value, and its descendants, as well as two methods:
- `is-BST` returns `True` if this node respects the rules according to its descendants;
- `is-BST-from-here` recursively tests all the tree and its subtree.

<br/>
Therefore, testing the tree is as simple as invoking the latter method on the root:

<br/>
<br/>
```raku
class Node {
    has Int $.value;
    has Node $.left is rw;
    has Node $.right is rw;

    method is-BST() {
        my $ok-left  = ! $!left  || ( $!left && $!left.value < $!value );
        my $ok-right = ! $!right || ( $!right && $!right.value >= $!value );
        return $ok-left && $ok-right;
    }

    method is-BST-from-here() {
        my $am-I-ok = self.is-BST();
        return False if ! $am-I-ok;

        my $ok-right = True;
        $ok-right = $!right.is-BST-from-here if $!right;
        my $ok-left  = True;
        $ok-left = $!left.is-BST-from-here if $!left;
        return $ok-right && $ok-left;
    }
}

sub MAIN() {
    my $root         = Node.new( value => 8 );
    $root.left       = Node.new( value => 5 );
    $root.left.left  = Node.new( value => 4 );
    $root.left.right = Node.new( value => 6 );
    $root.right      = Node.new( value => 9 );

    "1".say if $root.is-BST-from-here;
}

```
<br/>
<br/>

At the end, the program prints `1` as requested by the task only if the tree is a balanced one.
