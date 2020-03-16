---
layout: post
title:  "Perl Weekly Challenge 52: random money and stepping numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 52: overlapping ranges and nobel numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 52](https://perlweeklychallenge.org/blog/perl-weekly-challenge-052/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation in Italy is ugly and to some extent desperate: it has been a week since I'm closed into my house with my son and my wife, we cannot move and cannot go outside. Probably the soldiers will become to monitor the streets very soon this week.
<br/>
I cannot go visiting my mum that lives 10 minutes by car from me, and it is not clear when this emergency status will give us a break.
<br/>
So, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine*, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 52 - Task 1

The first task was about *Stepping numbers*, numbers where the difference between single adiacent digits is only one unit.
<br/>
The first approach I thought about was to iterate over a sequence, disassembling every number into its single digits and then comparing every digit. Then, I realized that I can do the same *forward*: fixed the first number, I can compute every following digit keeping a difference of `+1` and `-1`. Therefore, my solution is:


```perl6
my ( $h, $d, $u ) = $from.comb;
my ( $H, $D, $U ) = $to.comb;
my @stepping = ();
while ( $h <= $H ) {

    for 1, -1 {
        my $dd = ( 0 <= $h + $_ <= 9 ) ?? $h + $_ !! Nil;
        for 1, -1  {
            my $uu = ( 0 <= $dd + $_ <= 9 ) ?? $dd + $_ !! Nil;

            @stepping.push( $h * 100 + $dd * 10 + $uu ) if ( $uu && $dd );
        }
    }

    $h++;
}

say @stepping.grep( $from <= * <= $to ).join( "\n" );
```

I disassemble the starting number `$from` and ending number `$to` into their own digits. Then I loop until I'm sure I've gone over the ending number. At every iteration I compute the following number adding `+1` and `-1` and do the same with the following number.
Therefore, starting from `100`, I got `$h = 1`, then I compute `$dd` as `1 + 1` and `1 - 1`, finally I compute `$uu` as `2 + 1` and `2 - 1`. The result provides me `101` and `123`. There are no more stepping numbers that begins with `1`, so I increment `$h` and strt with `2`, and so on.



<a name="task2"></a>
## PWC 52 - Task 2

Task 2 was not clear to me: it was a play againt the computer in the search of picking more valuable moneys from a set on the table.
<br/>
Assuming I want to cheat, and made myself win (as if I was really playing), the player will always choose first the money with the highest value. So:

```perl6
while ( @moneys.elems ) {
    my $left  = @moneys.shift || 0;
    my $right = @moneys.pop   || 0;

    @player.push:   $left > $right ?? $left !! $right;
    @computer.push: $left > $right ?? $right !! $left;
}
```

Once the player has done his move, the computer must do the opposite.
<br/>
If we want to simulate a random challenge, the program becomes even more simple:


```perl6
while ( @moneys.elems ) {
    if 99.rand.Int %% 2 {
        @player.push:   @moneys.shift || 0;
        @computer.push: @moneys.pop   || 0;
    }
    else {
        @player.push:   @moneys.pop   || 0;
        @computer.push: @moneys.shift || 0;

    }
}
```

At every turn a random number is picked and so the player picks from the left or the right of the list and the computer does the opposite.
<br/>
I suspet the intent of the task was to consider all the possibilities, that make me think a *tree* structure has to be built.
