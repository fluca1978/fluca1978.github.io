---
layout: post
title:  "Perl Weekly Challenge 71: linked lists and peaking arrays"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 71: linked lists and peaking arrays

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 71](https://perlweeklychallenge.org/blog/perl-weekly-challenge-071/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
There is a lot of pressure about the schools, since we are approaching the new year for students and people like me, who have a son of around ten years old, are worried about the schools. Quite frankly I'm not worried about the virus per-se, rather the fact that the schools could not open or could introduce some fuzzy approach to lessons like splitting classes, reducing the timetable and so on.
<br/>
Besides, our situation due to the coronavirus is pretty much the same as the last week.

### Olivia
Olivia has started to sleep again on her catwalk, that connects the balcony to a great tree in the garden and is therefore at around 3 meters from the ground.
<br/>
I really admire her while staying, calm and confident, at such an high with no particular problem.
<br/>
Besides this, she is still walking with a lot of difficulties and apparently has some pain.


### Eyes

Something it is really hard to not think about, and something that is keeping me up at night.
Besides, no update so far.

<a name="task1"></a>
## PWC 71 - Task 1

The first task was quite simple after all: you have to *peak* an array of self-generated random numbers.
I decided to implement in the following way:

<br/>
<br/>
```perl6
sub MAIN( Int $N where { $N > 1 } ) {
    my @array;
    my @peak;

    while ( @array.elems < $N ) {
        my $random = ( 1 .. 50 ).rand.Int;
        @array.push: $random if ( ! @array.grep( * ~~ $random ) );
    }


    for 0 ..^ @array.elems {
        my $neighbour-left  = $_ == 0 ?? Nil !! @array[ $_ - 1 ];
        my $neighbour-right = $_ == ( @array.elems - 1 ) ?? Nil !! @array[ $_ + 1 ];
        my $current         = @array[ $_ ];

        @peak.push: $current if ( $neighbour-left && $current > $neighbour-left )
                             || ( $neighbour-right && $current > $neighbour-right );
    }

    "Array: { @array }".say;
    "Peak:  { @peak  }".say;
}

```
<br/>
<br/>

The first `while` loop generates random integers tied to a range, and push the numbers into the array only if they are not there already. Therefore the usage of `grep` ensures the array is made by unique numbers.
<br/>
The `for` loop analyzes every element of the array and searches for the left and right neighbours of the current element. This is something not clear in the specification of the task, so I decided to evaluate both sides of an element. If an element is greater than its left or right sibling, it is pushed to the `@peak` array. Of course, changing this last condition makes it possible to evaluate something different.

<a name="task2"></a>
## PWC 71 - Task 2

The second task required to remove an *nth* element from the end of a *simple linked list*.
First of all, I define a linked list `Node` with utility methods to comute the `size` of the list from this node and to print it out.

<br/>
<br/>
```perl6
class Node {
    has $.value;
    has $.next is rw;


    method Str() {
        my $current = self;
        my $string;
        while ( $current ) {
            $string ~= "{ $current.value }";
            $string ~= " -> " if $current.next;
            $current = $current.next;
        }

        $string;
    }


    method size() {
        my $size-of-the-list = 0;
        my $current = self;
        while ( $current ) {
            $size-of-the-list++;
            $current = $current.next;
        }

        $size-of-the-list;
    }
}

```
<br/>
<br/>


<br/>
<br/>
```perl6
sub MAIN( Int $N where { $N > 0 }, *@list ) {
    say @list;

    # build the list backward and keep the root
    # at the list
    my $root = Nil;
    my $current = Nil;
    loop ( my $i = @list.elems - 1; $i >= 0; $i-- ) {
        $root = Node.new( value => @list[ $i ],
                          next => $root );
    }

    # compute the size of the list
    my $size-of-the-list = $root.size;
    "Size of the list is $size-of-the-list".say;
    $root.put;





    my $index = 1;
    my $index-to-remove = $size-of-the-list - $N + 1;


    # particular case: remove the root
    $root = $root.next if ( $index == $index-to-remove || $N > $size-of-the-list );

    # remove a specific element, but only if this has not been already done
    # by removing the root element and thus changing the size of the list
    if ( $root.size == $size-of-the-list ) {
        $current = $root;
        while ( $current ) {
            if ( ( $index + 1 ) == ( $size-of-the-list - $N + 1 ) ) {
                $current.next = $current.next.next;
                last;
            }

            $index++;
            $current = $current.next;
        }
    }


    # print the modified list
    $root.put;

}

```
<br/>
<br/>

Then I accept an array `@list` and convert it to a linked list by walking it backward, so to place the forward references on each node. At the end, the `$root` node is the head of the list.
<br/>
I then compute the size of the list and the index to remove from the end of the list, and this is just for convenience.
<br/>
Then it comes a special case: the root/head has to be removed if the index to remove is equal to `1` or greater than the list size.
<br/>
Otherwise, if the current size of the list did not change, that means no special case has been applied, I walk the list forward and see if the *next-to-current* element is the one to be removed. In the case it is the one to be removed, it is simple enough to exclude it from the list by adjusting the `next` link to the `next.next` element, terminating the loop with a `last`. Otherwise, I move forward.
<br/>
It is worth noting that the above loop could have been inserted into a class method, so that the code results a lot more readable.
