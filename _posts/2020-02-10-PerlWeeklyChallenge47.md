---
layout: post
title:  "Perl Weekly Challenge 47: Roman Number Calculator and Gapful Numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 47: Roman Number Calculator and Gapful Numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 47](https://perlweeklychallenge.org/blog/perl-weekly-challenge-047/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<a name="task1"></a>
## PWC 47 - Task 1

The first task required to write a Roman Number calculator, so something that can accept an expression on the command line using Roman Numbers.
The problem had to be split into two main functions: one to decode a roman number into arabic values, and one to do the viceversa.
Let's see how to convert a roman number into an arabic one:

```perl6
my %roman-to-arabic = :I(1), :V(5), :X(10), :L(50), :C(100), :D(500), :M(1000);

sub convert-roman-to-arabic( Str:D $roman  ) {
    # convert to uppercase, reverse the string
    # and translates into numbers. For example,
    # IX => [ 10 1 ]
    # then map so that the current value is multiplied by -1 if the
    # previous one is higher
    my @arabic-digits = $roman.uc.comb.reverse.map: {
        state $last = 0;
        my $value = %roman-to-arabic{ $_ };
        $value *= -1 if $value < $last;
        $last = $value;
        $value;
    };

    return [+] @arabic-digits;
}
```

The trick I used is quite simple:
1) I convert the string into upper case so that each letter can be found as a key into the `%roman-to-arabic` hash of values;
2) I reverse the list of letters, so that `IX` becomes `XI`;
3) I walk the list of letters and, if the last seen value is greater than the current one it means that I have to subtract (that is the value must be modified to a negative number);
4) I then perform the sum of all the values and I obtain the value.

<br/>
To better explain, imagine I've got `VII`: I reverse it to `IIV` and then compose an array of number `[1 1 5]` and finally I sum to get `7`. In the case of `IX` I reverse it to `XI` and to the array `[10 1 ]` but since the last seen value when I encounter `1` is `10`, the number has to be negated, so `[ 10 -1 ]` that points me to the final value `9`.
<br/>
<br/>
More complicated is the function to do the opposite:

```perl6
sub convert-arabic-to-roman( Int $arabic  ) {

    # special cases: what to subtract from every value
    my %subtractors = 1_000 => 100,
    500 => 100,
    100 => 10,
    50 =>  10,
    10 =>  1,
    5 =>  1,
    1 => 0;

    # reverse the map of the values so that each arabic value
    # corresponds to a letter
    my %translator = %roman-to-arabic.map( { $_.value => $_.key } );


    return '' if ! $arabic;

    for %subtractors.pairs.sort: { $^b.key.Int <=> $^a.key.Int } -> $pair {
        my $subtractor = $pair.key.Int;
        my $removing   = $pair.value.Int;

        return %translator{ $subtractor } ~ convert-arabic-to-roman( $arabic - $subtractor.Int ) if $arabic >= $subtractor;
        return %translator{ $removing } ~ convert-arabic-to-roman( $arabic + $removing.Int ) if $arabic >= $subtractor - $removing;

    }

}
```

First of all, I need another hash that contains special *subtractions*, like `IX` that means that I can subtract `1` from `10` and therefore the number `10` has a subtracting value of `1`, here I build the map `10 => 1`.
I also need to reverse the `%roman-to-arabic` hash from *letter-to-value* into a *value-to-letter* hash, so I re`map` it to a different hash.
Then I sort the subtracting set of values from the greatest to the lowest and see if the current number that I have is greater then current value, if so I need to add the corresponding letter and do the same with the remaining part. If not, I have to consider the subtraction part (`$removing`) and append such letter to the letter found for the greater part.
<br/>
<br/>

Last, the script:

```perl6
my $operand-a = convert-roman-to-arabic( @*ARGS[0] );
my $operand-b = convert-roman-to-arabic( @*ARGS[2] );

my $result = do given @*ARGS[1].trim {
    when '+' { $operand-a + $operand-b; }
    when '-' { $operand-a - $operand-b; }
    when '/' { $operand-a / $operand-b; }
    when '*' { $operand-a * $operand-b; }
};

say convert-arabic-to-roman( $result );
```

The first and last arguments are converted from romant to arabic, then a `given` switch decides which computation to perform and place it into `$result`, that is lastly converted to roman and printed.
I admit this is not the most robust calculator, but seems to work to different tests. There might be edge cases that I've not thought about.


<a name="task2"></a>
## PWC 47 - Task 2

In this task I need to print the first 20 Gapful numbers, that are described as:
```
# Gapful numbers >= 100: numbers that are divisible by the number
# formed by their first and last digit.
# Numbers up to 100 trivially have this property and are excluded.
```

This is somehow simple: I looped from `100` to infinity, pushing a number into an array if it is a Gapful number, and stopping the loop when the array has more than 20 elements.

```perl6
for 100 .. Inf {
    $_ ~~ / ^ $<first>=\d \d+ $<last>=\d $ /;
    my $divisor = ( $/<first> ~ $/<last> ).Int;
    @found.push: $divisor if $_ %% $divisor && ! @found.grep: { $_ == $divisor };
    last if @found.elems == $limit;
}
```

With a regular expression I got the first and last digit, and then I see with `%%` if the number is divisible by such number, and if so I add it to the `@found` array but only if that number is not already present. Here `$limit` is a `MAIN` argument set to 20 by default.
