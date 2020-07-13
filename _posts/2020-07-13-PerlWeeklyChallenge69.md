---
layout: post
title:  "Perl Weekly Challenge 69: Strobogrammatic and 0-1 numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 69: Strobogrammatic and 0-1 numbers

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
While I'm still having a good mood about a possibile solution to the situation, the italian government is going to extend the state of emergency up to the end of the year, which seems to me quite strange at least with regard to other countries in the europe.


### Olivia
Olivia did her catwalk to the kitchen tree twice in the last week!
<br/>
This is somehow comfortating, evdoes not seem to be confindent enough to stay there.


### Eyes

I'm really frustated about my sight, and quite sad, and this, as you can imagine, is having a very bad influence in my personal life. We have scheduled the very next check up for the end of the following week.

<a name="task1"></a>
## PWC 69 - Task 1

The first task was about finding out Strobogrammatic numbers, that are numbers you can read upside down, or better *down on the upside* (if you love *Soundgarden*).
<br/>
I've decided to brute force implement them using a `gather/take` approach, since the list of numbers can be quite large (even if the program is going to print them all, thus vanishing the effort of the `gather/take`).
The idea is to have an hash that contains mappings between digits that can move upside down, for example `6` with `9` and, on the other side, `9` with `6`. The program then checks if a number has the leftmost number that is an upside-down of the rightmost, and in the case it is, moves to the leftmost plus one number and checks against the rightmost minus one, and so on. There are some particular edge cases, for example a number with an odd count of digits must have a central digit that is a *self upside down number* like `1`, `0` or `8`; this is a kind of short-circuit in the test.
<br/>
Therefore the application results in:

<br/>
<br/>
```perl6
sub MAIN( Int $A where { 1 <= $A <= 10**15 } = 1
          , Int $B where { $A <= $B <= 10**15 } = 10**15 ) {
    say "Working from $A to $B";

    my %reverse = 0 => 0
    , 1 => 1
    , 6 => 9
    , 8 => 8
    , 9 => 6;


    my @found = gather {
        for $A .. $B {
            # special case: single number
            take $_  if $_.chars == 1 && $_ == any( 0, 1, 8 );

            my @digits = $_.split: '', :skip-empty;

            # special case: if the number of digits is odd
            # the central digit must be a self reversing one
            next if ! @digits.elems %% 2
            &&  @digits[ ( @digits.elems / 2 ).Int ] != any ( 0, 1 , 8 );

            my $ok = True;
            CHECKING:
            for 0 ..^ @digits.elems -> $index {
                my ( $left, $right ) = $index, @digits.elems - $index - 1;
                last if $left == $right || $left > $right;
                $ok = False if ( %reverse{ @digits[ $left ] }:!exists )
                || ( %reverse{ @digits[ $right ] }:!exists );

                last CHECKING if ! $ok;
                $ok &= %reverse{ @digits[ $left ] } == @digits[ $right ];
                last CHECKING if ! $ok;
            }

            take $_ if $ok;
        }
    } # end of gather


    @found.unique.join( ', ' ).say;

}

```


<br/>
<br/>
I used junctions to quickly test for single-digit numbers and odd-counting numbers. In the case a number has to be deeply tested, the `for` loop performs the check with the `$left` and `$right` indexes to move across the digits. First of all, if one of the digits does not appear in the hash, the number cannot be made upside down, so I quit the loop. Then, I simply checks if the upside down value of the left digit is equal to the right digit, and as soon as I find a mismatch I terminate the loop.
<br/>
In the end, the `@found` array is printed.

<a name="task2"></a>
## PWC 69 - Task 2

The second task required to implement two particular functions, `switch` and `reverse` to manipulate strings made by only `1` and `0` characters.
<br/>
The function I implemented are as follows, even if I could have been done a better job with regular expressions:

<br/>
<br/>
```perl6
sub switch( Str $string where { $string ~~ / ^ <[0 1]>+ $ / } ) {
    my @bits;
    for $string.split( '', :skip-empty ) {
        @bits.push( 0 ) && next if $_ == 1;
        @bits.push: 1 if $_ == 0;
    }

    @bits.join;
}


sub reverse( Str $string where { $string ~~ / ^ <[0 1]>+ $ / } ) {
    $string.split( '', :skip-empty ).reverse.join;
}
```
<br/>
<br/>

Then the program has to build up a string with a certain limit of iterations so that a certain value is made by its precedessor mangled by the above functions.
<br/>
Initially I thought an array would helped solving the problem, but since you are requested to refer always to the very last generated value, a single scalar can help keeping saving some memory:


<br/>
<br/>
```perl6
sub MAIN( Int:D $max? = 100  ) {
    my $current-string;
    for 0 .. $max {
        $current-string = ''  if $_ == 0;
        $current-string = '0' if $_ == 1;
        next if $_ <= 1;
        $current-string = $current-string ~ '0' ~ switch( reverse( $current-string ) );
    }

    $current-string.say;

}
```
<br/>
<br/>


It is worth noting that passing a `100` maximum value for the iterations makes my computer to work for around 5 minutes without producing any result, and then I stopped it! Even a "simple" value of `10` makes a string `1024` chars in length!
