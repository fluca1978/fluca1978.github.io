---
layout: post
title:  "Perl Weekly Challenge 135: in a rush"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 135: in a rush

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 135](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0135/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 135 - Task 1

The first task was about finding the three middle digits in a given number, assuming it has an odd number of digits. This is implemented as:

<br/>
<br/>
```raku
sub MAIN( Int $n ) {
    $n.abs.say and exit if $n.Str.chars == 3;
    'too short'.say and exit if $n.Str.chars < 3;
    'even number of digits'.say and exit if $n.Str.chars %% 2;

    my $remove = ( $n.Str.chars - 3 ) / 2;
    $n.Str.comb[ $remove ..^ $remove + 3 ].join.say;

}

```
<br/>
<br/>

The `$remove` variable contains how many character it is need to remove from the left and right of the digit, and then I `comb` the input digit and slice it, re-`join` and print.



<a name="task2"></a>
## PWC 135 - Task 2

The second task was about validating SEDOL strings. It took me more to understand the alghoritm than to implement it.

<br/>
<br/>
```raku
sub MAIN( Str $SEDOL ) {

    my %map-of-digits;
    for flat 0 ..9, 'A' .. 'Z' {
        %map-of-digits{ $_ } = $++;
    }

    my @weights     = 1, 3, 1, 7, 3, 9;
    my @values      = %map-of-digits{ $SEDOL.comb };
    my $sum         = [+] @values Z* @weights;
    my $check_digit = (10 - $sum % 10) % 10;

    "1".say and exit if $SEDOL ~~ / $check_digit $ /;
    0.say;

}

```
<br/>
<br/>

The `%map-of-digits` is an hash where each digit is used as key, and the value is the digit to be used in the sum. The `@values` extracts the values to be summed using the `comb` of the input string, so each key in the hash.
Then summatory is made using the `[+]` reduction operator on the zip multiplication of each value for its weigth.
The the `$check_digit` is computed as the alghoritm wants. Last, if the input string is ending with the check digit, the program prints `1` and exits, otherwise it prints `0` and terminates.
