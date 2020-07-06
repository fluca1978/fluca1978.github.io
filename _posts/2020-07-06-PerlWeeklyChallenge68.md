---
layout: post
title:  "Perl Weekly Challenge 68: matrixes and linked lists"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 64: matrixes and linked lists

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 64](https://perlweeklychallenge.org/blog/perl-weekly-challenge-064/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
We are experiencing another increase in the number of hospitalized people.
<br/>
I'm pretty optimistic about the possible solution, but it is still a strange context to live into.


### Olivia
The vet said she has recovered a lot, and probably we need to wait another three months for see her jumping.
<br/>
We really hope, let's say it is strange to have her inside house all the day during summer.


### Eyes

This is a realy mind-brain.
<br/>
The last check emphasized that the pressure is still very high, so we decided to increase again the medications and to proceed with a laser therapy that, quite frankly, I don't thinkg will produce any result.
<br/>
It is almost sure I'm going to be hospitalized, the matter now is *when*.

<a name="task1"></a>
## PWC 68 - Task 1

The first task was about *zeroing* columns and rows of a matrix where a zero already exists: if a row or column has a zero, then zero all elements in the same row and column.
<br/>
Instead of working with a matrix, I assumed the matrix would come from the command line as a flat array. This makes a little harder to compute the offset for each element in rows and columns.
Anyway, instead of doing a brute-force loop over all the cells, I decided to `grep` the array for zeros and keep asde the indexes. Those indexes are absolute (the matrix is a one dimension array) and need to be converted into `$row` and `$column`, then it is quite simple to zero the row and column in the array.

<br/><br/>
```perl6
sub MAIN( Int $m, Int $n, *@incoming-matrix  ) {
    my @matrix = @incoming-matrix;
    "Original matrix was ".say;
    print-matrix( @incoming-matrix, $m, $n );
    my @zeros = @matrix.grep( * == 0, :k );

    for @zeros -> $zero-at {
        my ( $row, $column ) = ( $zero-at - 1 / $m ).Int, ( $zero-at % $n ).Int;
        @matrix[ $_ + $row ] = 0  for 0 ..^ $m;  # zero the rows
        @matrix[ $column + ( $_ * $m ) ] = 0  for 0 ..^ $n; # zero the columns
    }

    "Transformed matrix is".say;
    print-matrix( @matrix, $m, $n );

}
```
<br/><br/>

The utility function `print-matrix` is as simple as follows:

```perl6
sub print-matrix( @matrix, $m, $n ){
    say "----" ~  "--" x $m;
    for 0 ..^ $m -> $row {
        print "| ";
        for 0 ..^ $n -> $column {
            print "@matrix[ ( $row * $m ) + $column ] ";
        }
        say " |";
    }
    say "----" ~  "--" x $m;
}

```
<br/><br/>

Invoking this program produces a result similar to the following one:

<br/><br/>
```perl6
% raku ch-1.p6 3 3 1 0 1 1 1 1 1 0 1 
Original matrix was 
----------
| 1 0 1  |
| 1 1 1  |
| 1 0 1  |
----------
Transformed matrix is
----------
| 0 0 0  |
| 1 0 1  |
| 0 0 0  |
----------
```


<a name="task2"></a>
## PWC 68 - Task 2

Task two required to re-order a linked list by popping the last element and placing after the first one, the second-to-last to be placed as the third one and so on.
First of all, I declared a very dummy `Node` class:

<br/><br/>
```perl6
class Node {
      has Str $.value is rw;
      has Node $.next is rw;
}
```
<br/><br/>

Nothing fancy or complex, just to work with the list. The list has been initialized with a loop as follows:

<br/><br/>
```perl6
sub MAIN() {
    # build the list
    my Node $root = Node.new( :value( "L0" ) );
    my Node $current-node = $root;
    for 0 ^..^ 10 {
        $current-node.next = Node.new( :value( "L$_" ) );
        $current-node = $current-node.next;
    }

```
<br/><br/>

Now that I have a list to work with, let's convert into a flat array following the current links:


<br/><br/>
```perl6
    # convert into an array
    my @nodes;
    $current-node = $root;
    while ( $current-node ) {
        @nodes.push: $current-node;
        $current-node = $current-node.next;
    }

```


<br/><br/>

Now it is possible to build a *clone array* where the elements are shifted by their position depending on an index that is tied to the length of the array itself:

<br/><br/>
```perl6
    # sort into another array
    my @new-nodes;
    for 0 ..^ @nodes.elems / 2 {
        "switching elements $_ and { @nodes.elems - $_ - 1 }".say;
        @new-nodes.push: @nodes[ $_ ];
        @new-nodes.push: @nodes[ @nodes.elems - $_ -1 ];

    }
```
<br/><br/>

In the end, `@new-nodes` will have the desired order of the nodes.
It is now turn to adjust the nodes with the new links following the array:


<br/><br/>
```perl6
    # now adjust linked references
    for 0 ..^ @new-nodes.elems - 1 {
        @new-nodes[ $_ ].next = @new-nodes[ $_ + 1 ];
    }

    @new-nodes[ @new-nodes.elems - 1 ].next = Nil;

```


<br/><br/>
And all the nodes have now the correct link to the switched elements.
