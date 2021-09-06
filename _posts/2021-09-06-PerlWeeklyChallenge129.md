---
layout: post
title:  "Perl Weekly Challenge 129: trees and sums" 
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 129: trees and sums

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
## PWC 129 - Task 1

The first task was about finding the distance from the *root* in a binary tree given a node tag (value). The tree, apparently is not a *full binary* tree, meaning that a value could be on the left or the right of a node without any regard of its value.
<br/>
The program results as follows:


<br/>
<br/>
```raku
class Node {
    has Node $.left is rw;
    has Node $.right is rw;

    has Int $.value;
    has Int $.level = 0;

    method is-leaf() { ! $!left && ! $!right }

    method find-from-here( Int $needle ) {
        return self if $!value == $needle;

        my $r = $!right.find-from-here( $needle ) if $!right;
        return $r if $r;
        my $l = $!left.find-from-here( $needle ) if $!left;
        return $l if $l;
        return Nil if self.is-leaf;

    }
}


sub MAIN( Int $needle = 6 ) {
    my $level = 1;
    my $root = Node.new( value => 1,
                         left => Node.new( value => 2, level => $level ),
                         right => Node.new( value => 3, level => $level ) );
    $root.right.right = Node.new( value => 4, level => ++$level );
    $level++;
    $root.right.right.right = Node.new( value => 6, level => $level );
    $root.right.right.left = Node.new( value => 5, level => $level );



    my $current = $root.find-from-here( $needle );
    say "Found $needle at distance { $current.level }" if $current;
    say "Not found $needle" if ! $current;
}

```
<br/>
<br/>
The whole program resides in the `find-from-here` method in the class `Node`: such method returns the `Node` object that has the same value of the searching for one, and this means a single node with a specific value can exist in the tree or the nearest one will be found.
<br/>
Each node is initialized with a *deep* `level` from the root, so once I found the right `Node` it is trivial to get the distance from the root.

<a name="task2"></a>
## PWC 129 - Task 2

The second task was about summing two linked list starting from the end (rightmost elements) even when the lists are unbalaced and keeping each sum value less than `10`.
<br/>
I created a `LL` class that implements a single node in the list, with the `next` field for tracking the rightmost element of the current one. 
<br/>
To do the sum I created a `pop-last` method that removes and returns the rightmost one element from the list. More in detail, the element is not removed but made un-`available`, but this is an internal detail. Therefore, I cycle for the `max` length of the two lists, that means on all common elements, placing a default `0` value where the element is absent in the list.
Then I do sum them usin a temporary variable `$sum` and a `$carry` depending on the final result.
The sum is placed into a `@sums` array to build the final linked list.
<br/>
The `print` method flies on the list to print it in the left-to-right order.


<br/>
<br/>
```raku
class LL {
    has Int $.value = -1;
    has Bool $.available is rw = True;
    has LL $.next is rw = LL;


    method length() {
        my $counter = 1;
        my $current = self;
        $counter++ && $current = $current.next while ( $current.next );
        return $counter;
    }


    method pop-last() {
        my $current  = self;
        return Nil if ! $current.available;

        while ( $current.available && $current.next && $current.next.available ) {
            $current  = $current.next;
        }

        $current.available = False;
        return $current;
    }

    method print() {
        print $!value;
        if ( $!next  ) {
            print " -> ";
            $!next.print;
        }
    }
}


sub MAIN() {
    my $L1 = LL.new( value => 1 );
    $L1.next = LL.new( value => 2 );
    $L1.next.next = LL.new( value => 3 );
    $L1.next.next.next = LL.new( value => 4 );
    $L1.next.next.next.next = LL.new( value => 5 );

    my $L2 = LL.new( value => 6 );
    $L2.next = LL.new( value => 5 );
    $L2.next.next = LL.new( value => 5 );

    say "L1 = " ~ $L1.length;
    say "L2 = " ~ $L2.length;

    my ( $sum, $carry ) = 0, 0;
    my @sums;
    for 0 ..^ max( $L1.length, $L2.length ) {
        my ( $a, $b ) = ( $L1.pop-last, $L2.pop-last );
        my $sum = $carry
                   + ( $a ?? $a.value !! 0 )
                   + ( $b ?? $b.value !! 0 );

        if ( $sum >= 10 ) {
            $carry = ( $sum / 10 ).Int;
            $sum %= 10;
        }
        else {
            $carry = 0;
        }

        @sums.push: $sum;
    }


    my $R;
    my $current;
    my $previous;
    for @sums {
        $current = LL.new( value => $_, next => ( $previous ?? $previous !! Nil ) );
        $previous = $current;
    }

    say $current.print;
}
```
<br/>
<br/>

After this implementation, I realized that it is possible to solve the same problem without the need for a specific class, that is using `List` or `Array` as data structure:

<br/>
<br/>
```raku
sub MAIN() {
    my @L1 = 1 , 2 , 3 , 4 , 5;
    my @L2 = 6 , 5 , 5;

    my @sums;
    my $carry = 0;

    for 1 .. max( @L1.elems, @L2.elems ) {
        my $sum = ( @L1.elems >= $_ ?? @L1[ * - $_ ] !! 0 )
                      + ( @L2.elems >= $_ ?? @L2[ * - $_ ] !! 0 )
                      + $carry;
        $carry = 0;
        ( $sum, $carry ) = ( $sum % 10, ( $sum / 10 ).Int ) if $sum >= 10;
        @sums.push: $sum;
    }

    @sums.join( ' -> ' ).say;
}

```
<br/>
<br/>
The implementation is by far more compact, but the implementation alghoritm is the same.
