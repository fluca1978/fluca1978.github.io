---
layout: post
title:  "Perl Weekly Challenge 133: in a rush"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 133: in a rush

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 110](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0110/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 133 - Task 1

The first task was about implementing a way to compute the square root of an integer without using any pre-made function, nor library. I just re-implemented in Raku the alghoritm you can find around to get an approximation of the `sqrt`:

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {
    $n.say and exit if $n == 1;

    my Int $current-solution = $n +> 1;  # divide by two
    my Int $next-solution    = 0;
    while ( $next-solution < $current-solution ) {
        $next-solution    = ( $current-solution + $n / $current-solution ) +> 1 if ! $next-solution;
        ( $current-solution, $next-solution ) = $next-solution,
                                                ( $next-solution + $n / $next-solution ) +> 1;
        
    }

    $current-solution.say;

}
```
<br/>
<br/>





<a name="task2"></a>
## PWC 133 - Task 2

The second task was about computing the first ten *Smith numbers*, numbers where the sum of the factors is the same as the sum of its digits.
<br/>
The code looks like:

<br/>
<br/>
```raku
# a lazy list of all prime numbers
my @PRIMES = grep {.is-prime}, 1..*;

# divide a number into a list of its own factors
multi do-factor( 1 ) { (1) }
multi do-factor( Int $n where { $n > 1 } ) {
    my $needle = $n;
    my @factors;

    for @PRIMES -> $current-factor {
        # stop if we got a bigger number
        last if $current-factor > $needle;

        # skip if the number is not a divisor of what we are searching for
        next unless $needle %% $current-factor;

        # if here, it is a good factor
        @factors.push: $current-factor;

        # compute the remainder
        $needle /= $current-factor;
    }

    
    @factors.sort;
    

}


# It is a smith number if the sum of the digits
# is the sum of the factors
sub is-smith-number( Int $n where { $n > 0 } ) {
    return $n.comb.sum == do-factor( $n ).sum;
}


sub MAIN( Int $limit where { $limit > 0 } = 10 ) {

    my @smith-numbers;
    for 1 .. Inf {
        next if ! is-smith-number( $_ );
        @smith-numbers.push: $_;
        last if @smith-numbers.elems == $limit;
    }

    @smith-numbers.join( "\n" ).say;
}

```
<br/>
<br/>

I did not implemented a lazy approach here, rather I do iterate until I've found 10 numbers.
The `is-smith-number` is the method that checks if a number is good or not, and it does rely on the multi method `do-factors` that returns a list of numbers (factors) for the given input integer.
<br/>
The `do-factors` method in turn exploits a pre-made list of prime numbers, and iterates to find out all the prime numbers that can be used to made up a given input.
