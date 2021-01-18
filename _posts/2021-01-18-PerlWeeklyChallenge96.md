---
layout: post
title:  "Perl Weekly Challenge 96: Levenshtein distance"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 96: Levenshtein distance

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 96](https://perlweeklychallenge.org/blog/perl-weekly-challenge-096/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




# My eyes...

No good news so far: last check-up emphasized there is nothing to do right now.
<br/>
And probably there will be nothing to do in the future to improve the eye-sigth quality.

<a name="task1"></a>
## PWC 96 - Task 1

The first task was simple enough to be a single line in Raku: reversing the words of a sentence (not the word content, rather the word position). I created a one liner that splits the sentence depending on the number of spaces and reverses the resulting array.

<br/>
<br/>
```raku
sub MAIN( Str $phrase = 'The Perl    Weekly Challenge' ) {
    $phrase.split( / \s+ / ).reverse.join( ' ' ).say;
}
```
<br/>
<br/>


<a name="task2"></a>
## PWC 96 - Task 2

This task was about [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance){:target="_blank"} between two strings. Quite frankly, I did not have enough time to fully understand the alghoritm, so I started blindly implementing the approach suggested into Wikipedia, then refactoring it to a more *Raku* approach.
<br/>
In particular:
- the nested loop suggested into the Wikipedia alghoritm has been substituted by a `X` (cross) operator;
- the `+ cost` part has been substituted by a single `1 + ` part;
- there are a couple of special cases where I directly compare the strings to avoid all the looping;
- there is the need to *bootstrap* the strings with an equal character to make the first operation alsways a match.

<br/>
Therefore my function to compute the distance is:

<br/>
<br/>
```raku
sub distance ( Str $start, Str $end ) {

    # special case: one string is nil
    return $start.chars if ! $end;
    return $end.chars   if ! $start;

    # special case: strings are the same
    return 0 if $start.lc ~~ $end.lc;


    # use a bootstrap character so that the
    # distance-matrix is always initialized with a
    # <none> operation
    my @start-chars = 'X', |$start.comb;
    my @end-chars   = 'X', |$end.comb;


    my @distance-matrix;
    @distance-matrix[ $_; 0 ] = $_ for ^@start-chars.end;
    @distance-matrix[ 0; $_ ] = $_ for ^@end-chars.end;




    for  ^@start-chars.end X ^@end-chars.end -> ( $i, $j ) {
        next if $i == 0 || $j == 0;

        @distance-matrix[ $i; $j ] =
                  @start-chars[ $i ] ~~ @end-chars[ $j ]
        ?? @distance-matrix[ $i - 1; $j - 1 ]   # no operation, keep the previous computation
        !!
        1 +   # add a cost of 'one' operation
        min (
            @distance-matrix[ $i - 1; $j ]   # insert
            , @distance-matrix[ $i; $j - 1 ] # delete
            , @distance-matrix[ $i - 1; $j - 1 ]  # substitute
        );
    }


    # the last computation contains the distance
    return @distance-matrix[ *-1; *-1 ];

}
```
<br/>
<br/>

And the `MAIN` program results simply as follows:

<br/>
<br/>
```shell
sub MAIN( Str $S1 = 'kitten', Str $S2 = 'sitting' ){
    "Computing distance between '$S1' and '$S2' ==> %d"
    .sprintf( distance( $S1, $S2 ) )
    .say;
}
```
<br/>
<br/>
