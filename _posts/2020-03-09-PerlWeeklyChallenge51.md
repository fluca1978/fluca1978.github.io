---
layout: post
title:  "Perl Weekly Challenge 51: colorful numbers and triplets"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 51: colorful numbers and triplets

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 51](https://perlweeklychallenge.org/blog/perl-weekly-challenge-051/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONA VIRUS

Due to the high pressure that my city is living, I'm not fully concentrated on the weekly challenge. However, I decided to give it a try because, quite frankly, Raku is something that I love and solving puzzles in Raku help relaxing myself.


<a name="task1"></a>
## PWC 51 - Task 1

The first task consisted in finding triplets of numbers which, summed together, provide a specified sum, that I guess is set to zero for this particular task.
<br/>
Since no better solution came to mind, I decided to go for a *triple nested loop*:


```perl6
# extract only the integers
my @L = @*L.sort.grep( * ~~ Int );

# loop over the array
loop ( my $first-start = 0; $first-start < @L.elems - 2; $first-start++ ) {
    my $first = @L[ $first-start ];
    loop ( my $second-start = $first-start + 1; $second-start < @L.elems - 1; $second-start++ ) {
        my $second = @L[ $second-start ];

        next if $second < $first;

        loop ( my $third-start = $second-start + 1; $third-start < @L.elems; $third-start++ ) {
            my $third = @L[ $third-start ];

            next if $third < $second;

            @triplets.push: Array.new( $first, $second, $third ) if ( $first + $second + $third == $target; );
        }
    }
}
```

What happens is that I first sort `@L` to get a naturally sorted list of numbers. This should speed up the alghortim. Then I start getting a number and then got the following two sequences to see if the sum is equal to zero. If that is the case, I add a new array made by the three numbers into a `@triplets` list.
Essentially, I choose the first number, then the right one as second, then the right one as third and see if the sum provides me the right result. If not, proceed moving only the third number unles the end of the list is found, and then move the second and repeat, and so on.
<br/>
Having the list ordered allows me to do the scan only towards right.




<a name="task2"></a>
## PWC 51 - Task 2

Colorful numbers are those number where the product of sequential combinations are unique.
<br/>
It is not clear to me if the numbers that made up the number must be different too, however I decided to implement this using an ad-hoc function:

```perl6
sub is-colorful( Int:D $number ) {
    my @digits   = "%03d".sprintf( $number ).split( '', :skip-empty );

    # short-circuit: if the three numbers are the same,
    # than it is not colorful
    return False if [==] @digits;

    # stores the products into an hash to count
    # how many times it appear
    my %products;

    # all the digits product
    %products{ [*] @digits }++;

    # products of all sequences
    for 0 ..^ @digits.elems  {
        next if $_ == @digits.elems - 1;
        %products{ [*] @digits[ $_ ..^ @digits.elems  ] }++;
    }


    # NOT CLEAR: are the single digits to be included?
    %products{ $_ }++ for @digits;

    return False if %products.values.grep: * > 1;
    return True;
}

```

First of all, I split the number into its digits and keep them into an array named `@digits`. Then I use the *reduce* operator to test if the digits are all equals, because that means that there cannot be an unique product.
Then I use a `%products` hash to count how many times a product appears: the product value is the key of the has, and the occurrencies is the value.
I use the reduction operator to build the product of all the digits, and then a simple loop to compute the product of all the sequencies. Last I add all the digits as their own.
<br/>
Now it is quite simple to see if a number is colorful: if the values into the hash have a single value counted more than one, then the number is not colorful.
<br/>
It is trivial to build a program to test a list of numbers:


```perl6
$_.say if is-colorful( $_ ) for @numbers;
```
