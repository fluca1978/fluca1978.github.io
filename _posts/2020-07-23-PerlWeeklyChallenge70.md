---
layout: post
title:  "Perl Weekly Challenge 70: flipping and greying"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 70: flipping and greying

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 70](https://perlweeklychallenge.org/blog/perl-weekly-challenge-070/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
We are experiencing another increase in the number of hospitalized people.
<br/>
While I'm still having a good mood about a possibile solution to the situation, the italian government is going to extend the state of emergency up to the end of the year, which seems to me quite strange at least with regard to other countries in the europe.


### Olivia
Olivia tried to jump on the bathroom furniture, failing miserably and failing to the floor.
<br/>
It has been a bad scene.


### Eyes

Surgery has been scheduled at around one month from now.
<br/>
My mind na dmy spirit are going from north to south.

<a name="task1"></a>
## PWC 70 - Task 1

The first task was about switching characters in a string depending on a switching counter and an initial offset.
The brute force implementation consist of splitting the string into a char array and looping to switch the elements. I use `$left-index` and `$right-index` as a way to reduce the cluttering, but the important part here is the multiple assignments that, unlike other languages, do not require a swapping temporary variable.


<br/>
<br/>
```perl6
sub MAIN( Str $S,
          Int $C where { $C >= 1 },
          Int $O where { $O >= 1 && $O >= $C && ( $C + $O ) <= $S.chars } ) {
    my $N = $S.chars;

    say "$S with $N chars, swap counter $C and offset $O";

    my @chars = $S.split( '', :skip-empty );
    for 1 .. $C {
        my ( $index-left, $index-right ) = $_ % $N, ( $_ + $O ) % $N;
        ( @chars[ $index-left ], @chars[ $index-right ] ) = @chars[ $index-right ], @chars[ $index-left ];
    }

    @chars.join.say;
}
```


<a name="task2"></a>
## PWC 70 - Task 2

The second task was about implementing the Grey code, something I did not have a look since the high school years.
<br/>
It was interesting to remember how to generate automatically the Grey code, that essentially is based on duplicating the previous width bit array and placing a `0` as prefix on old values, and `1` on new values.
<br/>
Therefore, the implementation looks like:

<br/>
<br/>
```perl6
sub MAIN( Int $N where { 2 <= $N <= 5 } ) {
    say "Grey sequence with width $N";

    my @sequence;
    @sequence[ 0 ] = 0.base( 2 );
    @sequence[ 1 ] = 1.base( 2 );

    for 2 .. $N {
        # duplicate the sequence
        @sequence.push: | @sequence.reverse;

        # add the zero or one to the elements
        for 0 ..^ @sequence.elems -> $mirroring {
            @sequence[ $mirroring ] = ( $mirroring >= @sequence.elems / 2 ?? '1' !! '0' ) ~ @sequence[ $mirroring ];
        }

    }

    @sequence.map( *.parse-base( 2 ) ).say;
}
```


The `@sequence` array contains the bit string representation, and the `parse-base` method provided by the `Str` class converts every value in its integer representation as requested by the task.
Duplicating the sequence of bits is as easy as doing a `push` of a `reverse` array, `slip` because we don't want to nest lists.


## Why not on Monday?

Usually I try to push my solutions on Monday, this week I was away from the computer because last Sunday it was my birthday, and on Monday I went to the swimming pool with my son.
