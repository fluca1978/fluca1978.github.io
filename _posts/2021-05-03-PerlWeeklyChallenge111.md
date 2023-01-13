---
layout: post
title:  "Perl Weekly Challenge 111: Words and Matrix"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 111: Words and Matrix

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 111](https://perlweeklychallenge.org/blog/perl-weekly-challenge-111/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 111 - Task 1

The first task was assuming you have a matrix where each rows has integers so that the leftmost one is the lowest integer in the row, while the rightmost one is the highest integer in the row.
<br/>
You were supposed to implment a *good searching* alghoritm to find out if a given number is contained in the matrix or not.
<br/>
In Raku this could be solved with a `grep` call on the matrix, which is by far much more efficient of any other alghoritm I can come up with. However, given the problem definition, I suspect the intent was to make the search aware of the row boundaries.
<br/>
Therefore, I decided to go over each row and `grep` the row *if and only if* the given number was between the leftmost and rightmost values:

<br/>
<br/>
```raku
sub MAIN( Int $needle = 39 ) {
    my @matrix = [  1,  2,  3,  5,  7 ]
                 , [  9, 11, 15, 19, 20 ]
                 , [ 23, 24, 25, 29, 31 ]
                 , [ 32, 33, 39, 40, 42 ]
                 , [ 45, 47, 48, 49, 50 ];

    "1".say and exit  if $_[ 0 ] <= $needle
                         && $_[ *-1 ] >= $needle
                         && $_.grep: $needle for @matrix;

    "0".say;
}
```
<br/>
<br/>


<a name="task2"></a>
## PWC 111 - Task 2
The second task was about finding the longest english words that have the letters already sorted.
<br/>
It was not an hard job, since the only thing I have to test is if the words is the same as the sorted set of chars. However, how to find out the longest ones? I placed every word in an array, that in turn was into an hash where the number of chars (i.e., the size of the word) was the key.
<br/>
Then I got the highest key value and printed out the list of words.


<br/>
<br/>
```raku
sub MAIN( Str $words-file-name = '/usr/share/dict/words' ) {
    my %words;

    %words{ $_.chars }.push: $_ if ( $_.fc ~~ $_.fc.split( '' ).sort.join )
                  for $words-file-name.IO.lines();

    %words{ %words.keys.sort[ * - 1 ] }.join( "\n" ).say;
}
```
<br/>
<br/>

Please note the usage of `.fc` in order to check every word in a case insensitive way.
