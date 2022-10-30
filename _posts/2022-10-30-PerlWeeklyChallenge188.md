---
layout: post
title:  "Perl Weekly Challenge 188: I have no time this week!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 188: I have no time this week!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 188](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0188/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 188 - Task 1

Given an integer value and a list of integers, find out all the pairs in the list those sum is divisible by the specified integer.

<br/>
<br/>
```raku
sub MAIN( Bool :$verbose = False, Int :$k, *@list where { @list.elems == @list.grep( * ~~ Int ).elems } ) {
    my @pairs;
    for 0 ..^ @list.elems -> $i {
        for $i ^..^ @list.elems -> $j {
            @pairs.push: [ @list[ $i ], @list[ $j ] ] if ( ( @list[ $i ] + @list[ $j ] ) %% $k );
        }
    }

    @pairs.join( "\n" ).say if ( $verbose );
    @pairs.elems.say;
}


 ```
<br/>
<br/>

I decided to collect the pairs into an array named `@pairs`, so that in `$verbose` mode I can print out what I found. Thanks to a simple nested loop over the indexes, it is possible to find out the pairs that satisfy the condition.


<a name="task2"></a>
## PWC 188 - Task 2

Given two integers, find out how many operations are required to drop them both to zero, given that only one itneger can change at a given time depending on which one is greater than the other.

<br/>
<br/>
```raku
sub MAIN( Int $x is copy where { $x > 0 },
          Int $y is copy where { $y > 0 },
          Bool :$verbose = False ) {

    my @status;
    my $step = 0;
    while ( $x > 0 && $y > 0 ) {
        $x = $x - $y and @status.push( [ $x, $y ] ) if $x >= $y;
        $y = $y - $x and @status.push( [ $x, $y ] ) if $y >= $x;
    }

    @status.push: [ $x, $y ] if ( $x + $y != 0 && any( $x, $y ) == 0 );

    @status.join( "\n" ).say if $verbose;
    @status.elems.say;
}

```
<br/>
<br/>

I saved every step and their values into the `@status` array, so that I can print out the evolution of values.
The only thing to note is that when the `while` ends, there could be the need to store the last pair of values if they are still not to zero.
