---
layout: post
title:  "Perl Weekly Challenge 55: Weaving and Flipping"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 55: Weaving and Flipping

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 55](https://perlweeklychallenge.org/blog/perl-weekly-challenge-055/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation in Italy is ugly and to some extent desperate: it has been a few weeks since I'm closed into my house with my son and my wife, we cannot move and cannot go outside. Probably the soldiers will become to monitor the streets very soon this week.
<br/>
<br/>
I cannot go visiting my mum that lives 10 minutes by car from me, and it is not clear when this emergency status will give us a break.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine*, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 55 - Task 1

The first task was about flipping any bit in a limited range of a bit string to get the longest sequence of ones.
<br/>
I took a brute force approach here, doing a nested loop to flip the bits and counting with `grep` how many ones I have. Please note that the longest bit string of ones I could expect is at least the length of the string minus one:


```perl6
my @src = $number.comb;
my $N = @src.elems;

my $wanted =  $N - 1;
say "Searching for at least $wanted ones...";

for 0 ..^ $N -> $L {
    my @working = @src;
    @working[ $L ] = @working[ $L ] == 1 ?? 0 !! 1;
   
    for $L  ..^ $N -> $R {
        @working[ $R ] = @working[ $R ] == 1 ?? 0 !! 1 if ( $L != $R );
        say "Found { @working } with L = $L and R = $R" if @working.grep(  * == 1  ).elems >= $wanted;
    }
    
    
}
```

<a name="task2"></a>
## PWC 55 - Task 2

the second task was somehow simpler to me: wave an array so that the first element is greater than the second, than in turn is smaller than the third, and so on.
First I wrote a simple function to check if a couple of numbers are in right wave:

```perl6
sub is-ok( Int:D $a, Int:D $b, Bool:D $greater ) {
    return $a >= $b if ( $greater );
    return $a <= $b if ( ! $greater );
}
```

Then I looped over all the permutations of the array to see if I can get a wave sequence of numbers:

```perl6
for @array.permutations -> @current {
    my $ok = False;
    my $test = True; # true = greater
    for 0 ..^ @current.elems - 1 -> $index {
        $ok   = is-ok @current[ $index ].Int, @current[ $index + 1 ].Int, $test;
        $test = ! $test;
        last if ! $ok;
    }

    say "Found wave { @current }" if $ok;
}
```
