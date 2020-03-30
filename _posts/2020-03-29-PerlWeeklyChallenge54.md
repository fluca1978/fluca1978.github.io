---
layout: post
title:  "Perl Weekly Challenge 54: Permutations and Collatz"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 54: Permutations and Collatz

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 54](https://perlweeklychallenge.org/blog/perl-weekly-challenge-054/){:target="_blank"}.
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
## PWC 54 - Task 1

The first task is about printing a specific n-th element of a permutation of a list of numbers. This is quite simple, thanks to the fact that Raku already includes a `permutation` method in a list tha provides all the possible permutations. Therefore, it does suffice to produce a list of individual digits, permutate on that and extract the final result based on the position.

```perl6
sub MAIN( Int:D :$n where { $n >= 1 },
        Int:D :$k where { $k >= 1 } ) {

    "Computing the {$k}-th permutation of $n".say;

    # get all the single digits
    # that can be permutated
    my @digits;
    for 1 .. $n {
        @digits.push: $_;
    }


    my @permutations = @digits.permutations.sort;
    say "does not exist" if ( $k >= @permutations.elems );
    say @permutations[ $k - 1 ];
}
```


<a name="task2"></a>
## PWC 54 - Task 2

The second task was about Collatz sequences: a math sequence to move, step by step, from a positive integer to 1.
Implementing the function is quite simple:

```perl6
sub collatz( Int:D $m ) {
    my @sequence;
    my $n = $m;
    while ( $n > 1 ) {
        if ( $n %% 2 ) {
            $n /= 2;
        }
        else {
            $n = 3 * $n + 1;    
        }

        @sequence.push: $n;
    }

    @sequence;
}
```

and producing a `@sequence` allows for counting also the elements in the list.
Therefore, the program by itself becomes really easy:


```perl6
sub MAIN( Int:D $m where { $m > 0 } ) {

    my @sequence = collatz( $m );
    # print the results
    @sequence.join( " → " ).say;

...
}
```

### Task 2 Extra Credit
The extra credit was asking to print out the numbers that produced the longest twenty sequences, that was quite simple thanks to the above `collatz` function:

```perl6
# extra credit
my %extra;
for 1 .. 100000 {
    %extra{ $_ } = collatz( $_ ).elems;
}

# sort by the length
# prints 20 most length sequences data
for %extra.sort( { $^b.value <=> $^a.value } )[0..20] -> $p {
    "Number {$p.key} produces a Collatz sequence of {$p.value} numbers length".say;
}
```

The idea is to compute all the sequences and store them in an hash keyed by the number we are computing the sequence on. Then, we sort the hash descending on its value and print out the first twenty pairs.
