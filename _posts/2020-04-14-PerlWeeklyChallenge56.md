---
layout: post
title:  "Perl Weekly Challenge 56: Trees and Distance"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 56: Weaving and Flipping

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 56](https://perlweeklychallenge.org/blog/perl-weekly-challenge-056/){:target="_blank"}.
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
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine*, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 56 - Task 1

The first task was about finding elements in a sorted array that provides a specific difference (or distance). I decided to try a brute force method: I doubly walk over the array excluding equal indexes (look at the cat's ears in the second loop), and print the indexes and values if the are distant as `$K`. I use a junction to test both the positive and negative distance, so that the array can be sorted in ascending or descending ways.

```perl6
sub MAIN( Int:D :$K,  *@N ) {
    say "Array { @N } and index $K";
    for 0 ..^ @N.elems -> $i {
        for $i ^..^ @N.elems -> $j {
        say "$i, $j ---> { @N[ $i ] } <-> { @N[ $j ] }"
            if ( @N[ $i ] - @N[ $j ] == any( $K, $K * -1 ) )
        }
    }
}

```



<a name="task2"></a>
## PWC 56 - Task 2

The second task involved binary trees, even if the example is not what I know about trees since there is no rational spread across values in leaves.
Anyway, not being very used to trees, and not having well understood how the [tree method](https://docs.raku.org/routine/tree){:target="_blank"} can help me here, I decided to build my very own `Node` class as follows:

```perl6
class Node {
    has Int:D  $.value = 0;
    has Node $.left is rw;
    has Node $.right is rw;
    has Node $.parent is rw;
    has Bool $.is-leaf = False;
}

```
<br/>
It is almost straightforward: a node could have a couple of children, a parent, and its own value. To keep it really simple, there's a specific attribute that indicates if the node is a leaf or not.
<br/>
<br/>
Now it is possible to populate the example tree:

```perl6
   my $root                = Node.new( :value( 5 )  );
    $root.left              = Node.new( :value( 4 ), :parent( $root ) );
    $root.right             = Node.new( :value( 8 ), :parent( $root ) );
    $root.left.left         = Node.new( :value( 11 ), :parent( $root.left ) );
    $root.right.left        = Node.new( :value( 13 ), :parent( $root.right ), :is-leaf );
    $root.right.right       = Node.new( :value( 9 ), :parent( $root.right ) );
    $root.left.left.left    = Node.new( :value( 7 ),  :parent( $root.left.left ), :is-leaf );
    $root.left.left.right   = Node.new( :value( 2 ), :parent( $root.left.left ), :is-leaf );
    $root.right.right.right = Node.new( :value( 1 ), :parent( $root.right.right ), :is-leaf );


    my @nodes = $root
        , $root.left             
        , $root.right            
        , $root.left.left        
        , $root.right.left       
        , $root.right.right      
        , $root.left.left.left   
        , $root.left.left.right  
        , $root.right.right.right;

```

I keep the nodes into a flat list because I want to find quickly the leaves, so that I can do:

```perl6 
    my @leaves = @nodes.grep( *.is-leaf );

```

and once I've the leaves, I can walk upwards to the root to find out if the sum of the path provides me the right `$target` sum:


```perl6
    for @leaves  {
        my @path = [ .value ];
        my $node = $_;
        @path.push: $node.value while ( $node = $node.parent );
        my $sum = [+] @path;
        say "Sum is $sum with the path { @path.reverse }" if ( $sum == $target );
    }
```

<br/>
Nothing really special: I re-arranged the data structure to get the paths witht he specified sum. I'm sure Raku does have some quickest trick to walk through trees, but I cannot find anything today about it.
