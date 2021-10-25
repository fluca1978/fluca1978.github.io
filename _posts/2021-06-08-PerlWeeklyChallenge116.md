---
layout: post
title:  "Perl Weekly Challenge 116: numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 116: numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 116](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0116/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 116 - Task 1
This task has been difficult to me: given a number you have to split it in all possible combinations where numbers become subsequent, that is they differ from the previous one by one. As an example, given `1234` you spli it in `1, 2, 3, 4` and every digit differs from the left one by one.
<br/>
This is my implementation:

<br/>
<br/>
```raku
sub MAIN( Int $N where { $N >= 10 } ) {
    my @digits = $N.split( '', :skip-empty );
    my $min-length = 1;

    my @numbers;

    my $i = 0;
    my $done = True;
    while $done && $i < @digits.elems {



        # first number ever
        @numbers.push: @digits[ $i ] if ! @numbers;
        my $current-number = @numbers[ * - 1 ];

        # compute available next numbers
        my @next-number    = $current-number + 1, $current-number - 1;

        # see if there is room for any of the next
        # numbers in the remaining array of digits
        $done = False;
        for @next-number {
            my $length = $_.Str.chars;
            if $i + $length < @digits.elems {
                my $current = @digits[ $i + 1 .. $i  + $length ].join.Int;
                if $current == $_ {
                    @numbers.push: $current;
                    $i += $length;
                    $done = True;
                    last;
                }
            }
        }
    }

    # all done
    say @numbers if $done;
}


```
<br/>
<br/>

I split the number into its `@digits`, and then start iterating so that I place the very first number in the list into `@numbers`,
Then I compute how could be the next number, by adding and removing `1`, and I search for the number in the subsequent list of available digits.
<br/>
For example, given `91011` is start with `9`, then I compute the next available that would be `10` or `8` and I see if after the `9` there is one or the other, I find `10`, so I compute the next one that will be `11` or `9` and search it again.
<br/>
However, there is no assurance the first number in the list must be one digit only, so this implementation is not fully working.


<a name="task2"></a>
## PWC 116 - Task 2

The second task was easier: find if a number is made by digits that, if summed, provide a pure square of a number.
<br/>
My implementation is:
<br/>
<br/>
```raku
sub MAIN( Int $N where { $N >= 10 } ) {

    my $sum = $N.split( '' ).map( { $_ * $_ } ).sum;
    say 1 and exit if $sum.sqrt == $sum.sqrt.Int;
    say 0;
}

```
<br/>
<br/>

I start splitting the number into its digits, then I map every digit to its square, and I sum the final list.
Next I check if the `sqrt` of the sum is the same as its `Int` value, that means the sum is a pure square.
