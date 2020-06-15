---
layout: post
title:  "Perl Weekly Challenge 65: sums and palindromes"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 64: sums and palindromes

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
It is almost a week that Olivia is sleeping with us on our bed, and this made us a little happier. Olivia is now walking around the house, but still not trying to jump on everything higher than the sofa. She needs a lot of time to recover.

### Eyes

I'm in doubt about what the medications are doing, and if they are working well. I'm very nervous about, but I have to wait for the next week to come and have another check with my doctor.


<a name="task1"></a>
## PWC 65 - Task 1

The first task was about finding all numbers that have a specified number of digits and with a sum of the digits that correspond to a specific value.
<br/>
I decided to implement the task with a `for` loop that spawns between the minimal value for the number of digits to the max values, assumign that the max value can be a number starting with the expected sum and all trailing zeros. In fact, assuming the sum `$S` is `4` and the number of digits `$N` are `2`, the numbers to seek within range from `10` to `40`, because every number greater than `40` cannot provide a sum of its digits that is `4`.
<br/>
Therefore, the `MAIN` results in:

```perl6
sub MAIN( Int $N where { $N > 0 },
          Int $S where { $S >= 1 && $S <= 9 } ) {


    my @found;
    "Searching numbers with $N digits that sum to $S between { 1 ~ ( 0 x $N - 1 ) } and { $S ~ ( 0 x $N - 1 ) }".say;
    for ( 1 ~ ( 0 x $N - 1 ) ).Int .. ( $S ~ ( 0 x $N - 1 ) ).Int {
        @found.push: $_ if ( [+] $_.split( '', :skip-empty ) ) == $S ;
    }

    @found.say;

}

```

The `for` ranges from `1` and traling zeros for the number `$N` of digits, to the `$S` and trailing zeros. Both range lower and upper bounds are converted to `Int` in order to make the `for` walk with a single step increment.
Within the `for` loop, I add the number to the `@found` array only if the sum of its digits is what the program is searching for.

## Mind the Precedence

In the first run, I wrote a line like the following:

```perl6
@found.push: $_ if  [+] $_.split( '', :skip-empty )  == $S ;
```

that did not worked at all (`@found` was empty). The reason was explained on the [irc channel by Dakkar](https://colabti.org/irclogger/irclogger_log/raku?date=2020-06-15#l103){:target="_blank"}: *the `==` operator was at an higher precedence than the reduction `[+]` one, and so it was "mangled" into the sum*. Adding parens to the reduction fixed the problem!





<a name="task2"></a>
## PWC 65 - Task 2

The second task was about finding non overlapping palindrome partitions of a string.
<br/>
Again, I implemented it with a double nested `for` loop. 
The outer loop is responsible for seeking the next partition as soon as the first one is found, the inner loop is responsible for finding the largest palindrome in the string at the current position.
It goes like this:


```perl6
my $last = -1;
for 0 ..^ $S.chars -> $start {
    # avoid overlapping
    next if $start <= $last;

    for $start + 1 ..^ $S.chars -> $end {
        my $string = $S.split( '', :skip-empty )[ $start .. $end ];
        if $string ~~ $string.reverse {
            @palindromes.push: $string;
            # avoid overlapping
            $last = $end;
            last;
        }

    }
}
```

The indexes `$start` and `$end` are the partition bounds, so move from `0` to the max of the string `$S`.
If the found string partition is palindrome, it is added to the `@palindromes` array and the inner loop is stopped. The `$last` variable points to the `$end` of the found partition, so that the outer loop restarts from the next position after the end of the current partition. These two combines together to avoid overlapping.
<br/>
Then printing the result is simple as:

```perl6
@palindromes ?? @palindromes.join( ',' ).say !! "-1";
```
