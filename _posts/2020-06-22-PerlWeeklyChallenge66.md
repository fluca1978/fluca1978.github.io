---
layout: post
title:  "Perl Weekly Challenge 66: mangling integers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 64: mangling integers

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
The situation here in Italy is slowly coming back to normal: several places have openened the last week and while we are still forced to stay at home as much as possible, we can move a little more freely.
<br/>
I have visited my mom a couple of other times in the last week, and she came to visit me too.
<br/>
I've worked from office a couple of times, but still I do the main activity from home.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


### Olivia
Olivia found her way to the garden, but we promptly blocked her since she cannot jump yet and we are scared she can get hurted.

### Eyes

No news yet, but my doubts about the medications persist. At the end of the week I will have a check.


<a name="task1"></a>
## PWC 66 - Task 1

The first task was about doing a division between integers without using the division operators.
I implemented brutally a progressive subtraction, as follows:


```perl6
sub MAIN( Int $M, Int $N ){
    # extract both the signs
    my ( $signM, $signN ) = $M > 0 ?? 1 !! -1, $N > 0 ?? 1 !! -1;
    my ( $numerator, $denominator ) = abs( $M ), abs( $N );


    my ($result, $remind) = 0, $numerator;
    while ( $remind >= $denominator ) {
        $result++;
        $remind -= $denominator;
    }

    # this is a sign multiplication, it has nothing to do
    # with computing the result
    $result *= $signM * $signN;
    "$M / $N = $result (remind $remind)".say;

}

```

First of all I store the signs of the numbers, then convert them as absolute values and perform the subtraction.
The final result depends on the signs, and that is the only one multiplication in the code, and I assume that is allowed. In the case the `$N` is greater than `$M` then the result is just the reminder, and the `while` loop is not activated at all.


<a name="task2"></a>
## PWC 66 - Task 2

The second task was about finding a power of a given number, without knowing the base.
I first implemented a function to consider, given a base, what the power could be: the function returns the power or `Nil` if not found.

```perl6
sub is-power( Int:D $what = 100, Int:D $base = 10 ) {
    # if power of 1 must be itself!
    return 0 if ( $what == 1 );

    # if the values are the same, the pow is 1!
    return 1 if ( $what == $base );

    # not a power!
    return Nil if ( ! ( $what %% $base ) );

    # if here, should compute the power
    my $pow-increment = $what > $base ?? 1 !! -1;
    my Int $pow = 1 * $pow-increment;
    my Int $result = $base ** $pow;
    while ( $what > $result ) {
        $pow += $pow-increment;
        $result = $base ** $pow;
    }

    return $pow if $what == $result;
    return Nil;

}
```

There are a couple of shortcuts, like special values of `$what` and `$base` that can lead to obvious powers.
Note also that, in the case the `$what` values cannot be divided without a reminder by `$base` there is `Nil` value.
The power can be incremented by a single unit with a sign, because if the base is greater than the number, the power must be negative.
<br/>
Havin created the function, the `MAIN` results in a brutal `for` loop:

```perl6
sub MAIN( Int :$N ){
    my $found = False;
    for 2 ..^ $N {
        my $power = is-power( $N, $_ );
        "$N can be expressed as $_ pow( $power )".say && $found = True if $power;
    }

    "0".say if ! $found;

}
```

The only doubt I have, is that the `$_` should not scan the whole `2 ..^ $N` range, rather it should be locked at least at `$N/2`, and this could be a little optimization.
