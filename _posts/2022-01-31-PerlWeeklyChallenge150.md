---
layout: post
title:  "Perl Weekly Challenge 150: Fibonacci & squares (again!)"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 150: Fibonacci and squares (again!)

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 150](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0150/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg) and a [CTE only solution](#task2pgb)





<a name="task1"></a>
## PWC 150 - Task 1

This task was about producing a sequence of *Fibonacci's words*, that is given two initial words (made by digits), concatente them as in a Fibonacci sequence. Moreover, print the `51th` digits of the word with 51 characters and exit.


<br/>
<br/>
```raku
sub MAIN( Str $a, Str $b where { $a.chars == $b.chars
                                 && $a ~~ / ^ <[0 .. 9]>+ $ /
                                 && $b ~~ / ^ <[0 .. 9]>+ $ / } ) {
    my @fibonacci-words = $a, $b;
    for 0 .. Inf {
        @fibonacci-words.push: @fibonacci-words[ * - 2 ] ~ @fibonacci-words[ * - 1 ];
        @fibonacci-words[ * - 1 ].substr( @fibonacci-words[ * - 1 ].chars - 2, 1 ).say and last if @fibonacci-words[ * - 1 ].chars - 1 == 51;
    }

    @fibonacci-words.join( "\n" ).say;
}

 ```
<br/>
<br/>

The idea is simple: I loop forever concatenating at each step the previous string with the previous-previous one.
Then, if the current string, that is the last one in the `@fibonacci-words` array is of the right length, I print the required char. Please note that there is the need to consider that `51` is not a valid number of char, rathern it should be `52` as chars and string length are not the same.


<a name="task2"></a>
## PWC 150 - Task 2

The second task was harder: find out all the numbers that are *freesquare integers*. A  freesquare integer is an integer that has no repeating prime factor.

<br/>
<br/>
```raku
sub factors ( Int $number ) {
    return $number if $number == 1 || $number.is-prime;

    my @factors;
    my $value = $number;
    for ( 2 .. $value div 2 ).grep( *.is-prime ) -> $candidate {
        while $value %% $candidate {
            @factors.push: $candidate;
            $value /= $candidate;
        }
    }

    return @factors;
}


sub MAIN( Int $limit = 500 ) {
    my %solutions;
    for 1 .. $limit {
        my @factors = factors( $_ );

        my $ok = True;
        for @factors -> $current-factor {
            if @factors.grep( $current-factor ).elems > 1 {
                $ok = False;
                last;
            }
        }


        %solutions{ $_ } = @factors if $ok;
    }


    "$_ = { %solutions{$_}.join( ' x ' ); }".say for %solutions.keys.sort;
}


```
<br/>
<br/>

The `factors` routine is a *classical* routine that returns the prime factors of a given number.
In the `MAIN` I do iterate up to the specified `$limit`, and for every value I extract the `@factors` list. Then I iterate on each number of the list to catch if it repeats more than one time, and in such case I discard the value and start over with a new value. If the number, on the other hand, has non repeating factors, I place it into an hash `%solutions` aside its factors, so that I can print out every detail.
