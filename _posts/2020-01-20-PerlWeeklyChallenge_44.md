---
layout: post
title:  "Perl Weekly Challenge 44: only 100 and 200 bucks"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 44: only 100 and 200 bucks

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 44](https://perlweeklychallenge.org/blog/perl-weekly-challenge-044/){:target="_blank"}.


## PWC 44 - Task 1

Given the `123456789` string, it is required to find all possible algebric sums that made up the result of `100`.
[My solution](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/perl6/weekly-challenge/pwc_44_1.p6){:target="_blank"} composes a *List of Seq*s that compose all the available combinations of digits and operators. That is obtained applying the *zip operator* (`Z`) to the lists `@operators` and `@digits` and than performing a *hyperoperator* with the cross string concatenation `X~`.
Having obtained all possible combinations of digits and operators, I then loop against the obtained strings and convert every single digit in an integer number, since `.Int` keeps track of the sign. Then it does suffice to sum every result and see if I come to the final result number of `100`:

```perl6
or [X~] map { .Slip },  (@digits Z @operators) {

    my $expression = .Str.subst( ' ', '', :g );

    my $sum = 0;
    for $expression ~~ m:g/ (<[- + ]>\d+)/ {
        $sum += .Int;
    }


    say "Expression [$expression] = $sum" if $sum == 100;
}
```

## PWC 44 - Task 2

This has puzzled me for a long time before I did come to a very simple solution. The problem was to find the shortest way to double or increment a number (`1`) to reach the final result of `200`.
<br/>
Initially I started considering 200 increments, than trying to group them in order to reduce them by doubling a subset (e.g., 50 icnrements are like 25 doubled). No way to get a right approach.
<br/>
Then it comes to my mind a simple solution: since the doubling has to be the shortest operation, check from the end result how many times I can reduce it by a factor of 2, and if not possible, place an increment in the operation list and try again after having subtracted one value.
Therefore, quite simply:

```perl6
# $current-val = 200 initially
while ( $current-val > $initial-value ) {

  @moves.unshift: '%s to get %d $'
        .sprintf( $current-val %% 2 ?? 'double' !! 'add one buck', $current-val );
  $current-val = $current-val %% 2 ?? $current-val / 2 !! $current-val - 1;

}
```

After that, it does suffice to print out the `@moves` array and count how many elements it has in order to get the requested result.
<br/>
In a previous implementation I resolved the problem with a `given` control flow, but I did not liked the repetition of the descriptive strings:

```perl6
while ( $current-val > $initial-value ) {
 given  $current-val %% 2 {
     when .so {
         @moves.unshift: 'double to get %d $'.sprintf( $current-val );
         $current-val /= 2;
     }
     when ! .so {
         @moves.unshift: 'add one buck to get %d $'.sprintf( $current-val );
         $current-val -= 1;
     }
 };
}
```

However, as you can see, while the `given` approach is the most readable (to me), the `?? !!` one is the most compact.
