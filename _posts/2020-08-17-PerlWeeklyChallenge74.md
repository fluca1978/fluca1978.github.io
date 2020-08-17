---
layout: post
title:  "Perl Weekly Challenge 74: couting"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 74: counting

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 74](https://perlweeklychallenge.org/blog/perl-weekly-challenge-074/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 74 - Task 1

The first task was about finding the element in array that appears more than the half size of the array.
It has been quite simple to implement it:

<br/>
<br/>
```raku
sub MAIN( *@array where { @array.grep( * ~~ Int ).elems == @array.elems } ) {
    my $N = @array.elems;
    my $majority = floor( $N / 2 );

    my %counting;
    %counting{ $_ }++ for @array;

    given %counting.pairs.map( { .value >= $majority ?? $_ !! Nil } ).grep( * ~~ Pair ).unique.head {
        when .so { .key.say; }
        default  { '-1'.say; }
    }
}
```
<br/>
<br/>

First of all, I ensure that all the parameters provided to the script are integers (since slurpy array cannot be typed, I need to ensure it with an awkward `grep`).
<br/>
The `$majority` variable holds the compute majority value, as by the problem defintion. It is there just to make the code a little cleaner.
<br/>
The `%counting` hash simply counts every number in the array how many times it appears.
<br/>
Then I use a `given` to simplify the long `Pair` extraction: for every `Pair` in the array, I `map` it to a list where every element is the `Pair` itself only if the `value` (i.e., the number of occurrences) is greater than `$majority`, otherwise I place a `Nil` placeholder. This gives me a list that is made by pairs and nulls, so I `grep` out everything is not a `Pair`, and get the first unique value in the list (there could be more than one value that satisfy the problem requirement!).
<br/>
If what I get is a `Pair`, as expected, I print the `key`, that is the number in the array. Otherwise, if there is no `Pair` at all, I print the special value `-1`.

<a name="task2"></a>
## PWC 74 - Task 2

I spent much more time trying to figure out what the problem was asking me to do than to implement it.
I was stucked at a point where I was able to run one of the test but not the other or viceversa.
However, the implementation I wrote is:


<br/>
<br/>
```raku
sub MAIN( Str $S where { $S.chars > 2 } ) {

    my @result;
    my %counting;
    my @chars = $S.comb( '', :skip-empty );

    for 0 ..^ @chars.elems -> $index {
        my $current-char = @chars[ $index ];



        # if the result array is empty
        # or the char is not in the array, it is
        # ok to push
        @result.push( $current-char ) && next if ! @result || ! @result.grep( * ~~ $current-char );


        # if here I need to search for the first rightmost
        # not repeating char so far
        %counting = %();
        %counting{ $_ }++ for $S.substr( 0 .. $index ).comb( '', :skip-empty );
        my $fnr = $S.substr( 0 .. $index )
                    .comb( '', :skip-empty )
                    .reverse
                    .grep( { %counting{ $_ }:exists && %counting{ $_ } == 1 } )
                    .first // '#';

        @result.push: $fnr;
    }

    @result.join.say;
}
```
<br/>
<br/>

The idea is simple: I split the string into single chars that I store in the `@chars` array, and then I loop other the array. Since I need to slice the array or substring, I use numberical indexes, and `$current-char` is what I'm on at every loop interation.
<br/>
The easy case is when the array of `@result` is empty (first iteration of the loop) or the char does not appear at all in the array: in such case I can insert the char and restart the loop from the `next` iteration.
<br/>
The hard case is when the `$current-char` is already in the `@result` array: here I need to find out the first non repeating char. The non repeating char must be searched in the substring of the original input string up to the current char, that is `$S.substr( 0 .. $index )`. I split the string into single chars and use the `%counting` hash to count every occurrency. Now I consider again the substring, I `comb` it into single chars, I reverse them to the the rightmost char first, then I `grep` the resulting list searching for all the characters that appear exactly one time, and then I take the `first` element. If nothing like that exists, the special char `#` is used.
<br/>
It is possible to remove the `grep` intermediate list extraction, since `first` accepts a callable to use as filter:

<br/>
<br/>
```raku
 my $fnr = $S.substr( 0 .. $index )
             .comb( '', :skip-empty )
             .reverse
             .first( { %counting{ $_ }:exists && %counting{ $_ } == 1 } )
              // '#';
```
<br/>
<br/>

