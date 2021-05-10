---
layout: post
title:  "Perl Weekly Challenge 112: Climbing the canonical path!"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 112: Climbing the canonical path!

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 112](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0112/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 112 - Task 1
The first task was about writing a script to make a given filesystem path *canonical*, that is removing all the spurious data such as `..`, `.`, replicated slashes, and so on.
<br/>
Raku `IO::Path` already has a way to make a path canonical, but it does it without trimming out `..`, while the task required to interpret such characters in making the path *smart*. For example: `/a/b/../d` should resolve in `/a/d` because `..` makes the system to go up one level, so removing the `b` descending.
<br/>
Therefore, I decided to implement it using a `split` approach:
- I do split the path according to its separator;
- add every part of the path to an array if and only if it is not empty (meaning a multiple slash has occurred) and is not `.`, meaning nothing has to be done;
- in the case the part of the path is `..` I have to trim out the last value of the already computed array of parts, that is how I go up one level.

<br/>
<br/>
```raku
sub MAIN( Str :$path ) {

    my @results;
    for $path.split( '/' ) {
        next if ! $_ || $_ ~~ '.';
        @results.push: $_ if ( $_ !~~ '..' );
        @results = @results[ 0 .. * - 2 ] if $_ ~~ '..';
        
    }

    ('/' ~ @results.join( '/' )).say;
}
```
<br/>
<br/>

Please note I've added an extra leading slash in front of the `say` line, so that the path is outputted as absolute.

<a name="task2"></a>
## PWC 112 - Task 2
The second task was about finding out the sum of `1` and `2` that can lead to a specific value, aslo known as *climbing stairs* problem.
<br/>
I've implemented in a very *unefficient* way:
- I create an array of all possible `1` and `2` that can lead to the sum of the given number;
- I do iterate on every possible iteration of such an array, and see how many elements I need to get the sum;
- I print out the resulting arrays only using `unique` values.
<br/>
Therefore it results as follows:

<br/>
<br/>
```raku
sub MAIN( Int $top where { $top > 1 } ) {
    my @haystack = (1 xx $top, 2 xx $top / 2).flat;
    my @solutions;

    # get all possible combinations
    for @haystack.permutations {
        # check every sub-array that gives me the sum
        for 0 ..^ $_.elems -> $index {
            @solutions.push: $_[ 0 .. $index ]  if $_[ 0 .. $index ].sum == $top;
        }
    }


    # print only unique values
    say @solutions.unique: as => { .Str };
    
}
```
<br/>
<br/>

This is really expensive, since it can quickly require a lot of permutations to compute. 
Therefore I decided to follow a different approach:
- I build the very first solution, that is made by only `1` up to the `$top` value;
- I then start using the solution as a string of `1`s and substitute every `11` with a `2`;
- I do compute all the possible permutations of every `2` in the list.
<br/>
Therefore the program is:

<br/>
<br/>
```raku
sub MAIN( Int $top where { $top > 1 } ) {
    my @solutions;
    @solutions.push: 1 xx $top;

    my $current-solution = 1 x $top;
    while ( $current-solution ~~ / 1 ** 2 / ) {
        $current-solution ~~ s/ 1 ** 2 / 2 /;
        $current-solution = $current-solution.split( ' ', :skip-empty  ).join( '' );
        @solutions.push: $_ for $current-solution.split( '', :skip-empty ).permutations.unique( as => { .Str.trim } );
        
        
    }
    
    
    for @solutions -> @current-solution {
        say "\nPossible solution:";
        "%d step%s ".sprintf( $_, $_ > 1 ?? 's' !! '' ).print if $_ > 0 for @current-solution;
        
    }

    say "";
    
}
```
<br/>
<br/>

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
