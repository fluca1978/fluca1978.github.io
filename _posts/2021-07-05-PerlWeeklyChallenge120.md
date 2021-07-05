---
layout: post
title:  "Perl Weekly Challenge 120: quick and dirt"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 120: quick and dirt

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 110](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0110/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 120 - Task 1

The first task was about rotating, or better exchanging, bits position in abinary representation of a number: an odd bit has to be exchanged with its even pair and viceversa.
I decided to do that with a temporary array where I store the exchanged bits:

<br/>
<br/>
```raku
sub MAIN( Int $N where { 0 < $N < 255 } ) {
    my @bits = '%08s'.sprintf( $N.base( 2 ) ).split( '', :skip-empty );

    my @rotated-bits;
    for my @bits -> $odd, $even {
        @rotated-bits.push: $even, $odd;
    }


    @rotated-bits.join.Str.parse-base( 2 ).say;
}


```
<br/>
<br/>

I store the binrary representation of the number in the `@bits` array, already expanded to an eight bit representation. I then extract all the couples of elements and exchange them into `@rotted-bits`, and finally I `join` and reconvert by `parse(2)` from binrary to decimal notation.



<a name="task2"></a>
## PWC 120 - Task 2

In the second task there was the need to compute the minimal angle between clock arms, given a string representing the time:

<br/>
<br/>
```raku
sub MAIN( Str $time where { $time ~~ / ^ \d ** 2 ':' \d ** 2 $ / } ) {
    my ( $hour, $minute ) = $time.split( ':', :skip-empty );

    $hour   %= 12;
    $minute %= 60;
    $minute /= 5;

    say abs( $hour - $minute )  * 30;
}

```
<br/>
<br/>

First of all, I do extract the numeric values for `$hour` and `$minute` from the string representing the time. After that, I do convert the time so that the values are always within the range `1`..`12` and `1`..`60`, and then I compute the value of the minutes by its offset of `5` minutes. This makes me find out what tick the `$hour` and `$minute` are pointing to.
<br/>
Having the ticks, it does suffice to multiple the difference for the angle between the ticks to get the result.
