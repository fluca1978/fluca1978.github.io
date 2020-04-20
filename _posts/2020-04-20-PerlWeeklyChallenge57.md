---
layout: post
title:  "Perl Weekly Challenge 57: I suck at doing trees!"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 57: I suck at doing trees

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 57](https://perlweeklychallenge.org/blog/perl-weekly-challenge-057/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation in Italy is ugly and to some extent desperate: it has been a few weeks since I'm closed into my house with my son and my wife, we cannot move and cannot go outside. Probably the soldiers will become to monitor the streets very soon this week.
<br/>
<br/>
I cannot go visiting my mum that lives 10 minutes by car from me, and it is not clear when this emergency status will give us a break.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 57 - Task 1

The first task involved flipping a binary tree.
<br/>
**I hate binary trees**, I was good only at university in doing trees.
<br/>
Anyway, I decided to model my trees out of its nodes:


```perl6
class Node {
    has Int  $.value;
    has Node $.left  is rw;
    has Node $.right is rw;
}
```

so that I can build the tree as follows:

```perl6
my Node $root     = Node.new( :value( 1 ) );
$root.left        = Node.new( :value( 2 ) );
$root.right       = Node.new( :value( 3 ) );
$root.left.left   = Node.new( :value( 4 ) );
$root.left.right  = Node.new( :value( 5 ) );
$root.right.left  = Node.new( :value( 6 ) );
$root.right.right = Node.new( :value( 7 ) );
```


<br/>
Then I created a `switch` sub that, given a node, flips the left ad right side:

```perl6
sub switch( Node $current-node is rw ) {
    return if ! $current-node
        && ! $current-node.left
        && ! $current-node.right;

    my ( $left, $right ) = ( $current-node.left, $current-node.right );
    $current-node.left  = $right;
    $current-node.right = $left;

    switch( $current-node.left )  if $current-node.left;
    switch( $current-node.right ) if $current-node.right;
}
```


The above routine is recursive: once it has switched the left and right part of the current node tries to do the same for the flipped parts' children.
<br/>
This work for a fully balanced tree, and should work also for an unbalanced tree, but I'm not sure this simple approach covers all cases.

### Bonus Question

The bonus question was to provide a way to print the tree in a good way.
<br/>
Again, **I suck in dong trees!**
<br/>
I provided a recursive `print` method that prints a list of nodes, from left to right, attempting to provide a decent line alignment and a subline that prints the decorators:


```perl6
sub print ( @nodes ) {
    return if ! @nodes;
    
    my $spaces = " " x 6 / @nodes.elems;
    my @children;
    my $line = "";
    my $subline = "";
    loop ( my $i = 0; $i < @nodes.elems; $i++ ) {
        next if ! @nodes[ $i ];
        $line ~= $spaces ~ $spaces x $i;
        $line ~= @nodes[ $i ].value;
        $subline ~= $spaces ~ $spaces x $i;
        $subline ~= "/ \\";
        @children.push: @nodes[ $i ].left, @nodes[ $i ].right;
    }

    say $line;
    say $subline;
    print( @children );
}

```

<br/>
The end result is something less than decent:


```shell
% raku ch-1.p6
      1
      / \
   3      2
   / \      / \
 7  6   5    4
 / \  / \   / \    / \
```

As you can see the spacing is not optimal and the result provides node separators out of place. There must be a better computation with the center of every digit, but I don't want to spend more on time on this, sorry.



<a name="task2"></a>
## PWC 57 - Task 2

The second task was about finding the shortest unique prefix within a list of words.
This was uite easy thanks to the `.classify` method: such method accepts a `Whateverable` block of code that allows for building an hash with the key the result of the code and the value the word that was used as an input.
<br/>
Therefore, the following routine does the trick:


```perl6
sub prefix( @words, $len ) {
    for @words.classify( *.substr( 0, $len ) ) {
        if $_.value.elems == 1 {
            say "Prefix { $_.key } (length = $len)";    
        }
        else {
            prefix( $_.value, $len + 1 );
        }
    }
}
```

Starting with a substring, with `$len` set to 1, I can extract an hash of all the words that begin with a letter. If the value of the hash has a single element, then the prefix is fine, otherwise the sublist can be evaluated recursively with the increased length.
<br/>
Therefore the following program:

```perl6
sub MAIN( ) {
    my @words =  [ "alphabet", "book", "carpet", "cadmium", "cadeau", "alpine" ];

    prefix( @words, 1 );

}
```


produces the following output:


```shell
% raku ch-2.p6
Prefix alph (length = 4)
Prefix alpi (length = 4)
Prefix cadm (length = 4)
Prefix cade (length = 4)
Prefix car (length = 3)
Prefix b (length = 1)
```
