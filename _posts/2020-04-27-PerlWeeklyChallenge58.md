---
layout: post
title:  "Perl Weekly Challenge 58: (almost) semantic version numbers and people lineup"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 58: (almost) semantic version numbers and people lineup

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 58](https://perlweeklychallenge.org/blog/perl-weekly-challenge-058/){:target="_blank"}.
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
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 58 - Task 1

The first task was quite simple to solve with a *brute force* attack: it was requested to sort version numbers considering dots and underscore (with a weight lesser than the dot).
<br/>
I decided to operate a string substition assigning a different integer value to a dot and an underscore. Therefore:

```perl6
sub MAIN() {
    my Str @test = '0.1', '1.1'
        , '2.0', '1.2'
        , '1.2', '1.2_5'
        , '1.2.1', '1.2_1'
        , '1.2.1', '1.2.1';


    for @test -> $v1, $v2 {
        my $v1n = $v1.Str.subst( '.', 9, :g ).subst( '_', '0', :g );
        my $v2n = $v2.Str.subst( '.', 9, :g ).subst( '_', '0', :g );
        say "Comparing $v1 vs $v2 -> " ~ do  given $v1n <=> $v2n {
            when More { 1 }
            when Less { -1 }
            when Same { 0 }
        };

    }
}
```

The `$v1n` and `$v2n` are *normalized* values where `.` is substituted with a 9 and `_` with a zero, so that `1.2_3` becomes `19203`.
Therefore I can let Raku compare the strings with the `<=>` numeric comparator and translate the result as requested from the problem into the range `1`, `0`, `-1`.
<br/>
Of course this solution has a problem: what if you compare two incomplete version numbers? For instance: `1` versus `1_0` or worst `1` versus `1_` makes the latter wins, while probably the values are to be considered the same. This can be solved adding a little more logic to check that the version numbers are always complete and removing any trailing underscore or dot, but since the problem was not asking for that and I was in a rush I did not implemented it.



<a name="task2"></a>
## PWC 58 - Task 2

The second task was a lot more harder, even to understand.
<br/>
I decided, again, to use a brute force approach: I created an hash to map every height to its requested tallers, then crated all the possible solutions using the `.permutations()` method and checked for the validity of the solution.
<br/>
First of all, every solution must start with a requested taller equal to zero, or no solution at all can be found. Than, every subsequent height must be compared with the slice of the array that precedes it to ensure that the number of tallers i exactly what has been stored in the hash.
This is a piece of cake thanks to `.grep()` and postfixed conditionals.
<br/>
It looks therefore like this:

```perl6
sub MAIN(){
    my @H = 2, 6, 4, 5, 1, 3;
    my @T = 1, 0, 2, 0, 1, 2;


    # build an hash to map heights to tallers
    my %HT;
    for 0 ..^ @H.elems {
        %HT{ @H[ $_ ] } = @T[ $_ ];
    }


    # evaluate all possible solutions
    for %HT.keys.permutations -> @solution {
        # the leftmost element must have a taller set to zero!
        next if %HT{ @solution[ 0 ] } != 0;

        # the solution is good if the number of tallers for
        # every element is equal to the values of tallers
        my $ok = True;

        for 1 ..^ @solution.elems {
            my $height  = @solution[ $_ ];
            my $tallers = %HT{ $height };
            $ok = False if @solution[ 0 .. $_-1].grep( * > $height ) != $tallers;
            last if ! $ok;
        }

        say @solution if $ok;
    }
}
```
