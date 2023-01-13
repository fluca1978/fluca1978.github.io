---
layout: post
title:  "Perl Weekly Challenge 113: sums and trees"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 113: sums and trees

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 113](https://perlweeklychallenge.org/blog/perl-weekly-challenge-113/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 113 - Task 1
The first task was quite simple to me: given a number and a digit we need to find out every number containing only one repetition of the digit that, summed together, provide the number given as argument.
<br/>
<br/>
```raku
sub MAIN( Int $N where { $N > 0 },
          Int $D where { $D >= 0 && $D.Str.chars == 1 } ) {
    given (1 ..^ $N).grep( * ~~ / $D /).sum {
        when $N { '1'.say }
        default { '0'.say }
    }
    
}
```
<br/>
<br/>
The idea is quite simple:
- I produce all available numbers between `1` and the given target number `$N`;
- I exclude from such list every number that does contain more than one digit `$D`;
- I compute the `sum` of such list;
- if such sum is equal to `$N` I do print `1`, else I print `0`.


<a name="task2"></a>
## PWC 113 - Task 2
The second problem was a little too complicated to me, since it involved *binary trees*. In particular, given a tree I was asked to substitute every node with the sum of the remaining nodes of the tree.
<br/>
The difficult was that, given a node, I needed to substitute it with the sum of all nodes up and down from the current node.
<br/>
I decided to implement a `Node` class with common attributes such as `$value` as its content, `$left` and `$right` as *pointers* to children.

<br/>
<br/>
```raku
class Node {
    has Int $.value;

    has  $.left is rw;
    has  $.right is rw;

    method sum() {
        my $sum = $!value;
        $sum += $_.sum if ( $_ ) for ( $!left, $!right );
        return $sum;
    }

    method map( &block ) {
        self.new: value => block( $!value ),
            left => $!left.map( &block ),
            right => $!right.map( &block );
    }

    method say() {
        "{ $!value }".say;
        "\t Left  = { $!left.value }".say if $!left;
        "\t Right = { $!right.value }".say if $!right;

        $!left.say if $!left;
        $!right.say if $!right;
    }

    
}
```
<br/>
<br/>

The `sum` method provides the sum of the whole part of the three starting from the current `Node`, so that given the root node I can compute the sum of the whole tree.
<br/>
The trick here was to implement a `map`-like method that accepts a block of code and executes it against the `sum` of the node and its children.
<br/>
In particular the `map` method creates a new `Node` instance, the one that is going to substitute the node. Such substitute has the value computed as the block of code passed to `map`, and then recursively does `map` on the children nodes.
<br/>
The program therefore is as follows:
<br/>
<br/>
```raku
sub MAIN() {

    my $root = Node.new:
    value => 1
        , left => Node.new( value => 2, left => Node.new( value => 4, left => Node.new( value => 7 ) ) )
        , right => Node.new( value => 3, left => Node.new( value => 5 ), right => Node.new( value => 6 ) );

    $root = $root.map: { $root.sum - $_ if $_  };
    $root.say;
}


```
<br/>
<br/>

When `map` is invoked the block of code is `{ $root.sum - $_ if $_ }`, so it computes the sum of the whole tree (`$root.sum`) and substracts the value of the current node value (`$_` is a number because `map` passes `$!value` to the block of code as topic).
<br/>
Recursively doing this makes the tree a new one with the nodes changed to the requested values.
<br/>
<br/>
*I hate tree!* 



There is some machinery to avoid empty spaces here and there, and also I placed the `unique`ness of the solutions up to the permutation level, so that the final array `@solutions` is already made by unique lists.
<br/>
The printing also evolved so that I can correctly print `step` or `steps`. The result is:


<br/>
<br/>
```shell
% raku ch-2.p6 5

Possible solution:
1 step 1 step 1 step 1 step 1 step 
Possible solution:
2 steps 1 step 1 step 1 step 
Possible solution:
1 step 2 steps 1 step 1 step 
Possible solution:
1 step 1 step 2 steps 1 step 
Possible solution:
1 step 1 step 1 step 2 steps 
Possible solution:
2 steps 2 steps 1 step 
Possible solution:
2 steps 1 step 2 steps 
Possible solution:
1 step 2 steps 2 steps 
```
<br/>
<br/>

And this is it.
