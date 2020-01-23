---
layout: post
title:  "Perl Weekly Challenge 43: rings and self descriptive numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 43: rings and self descriptive numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 43](https://perlweeklychallenge.org/blog/perl-weekly-challenge-043/){:target="_blank"}.


## PWC 43 - Task 1

Essentially, you have to find the pair number out of a sequence to provide always the sum of 11.
[My solution](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/perl6/weekly-challenge/pwc_43_1.p6){:target="_blank"} creates an hash of the rings, that is which values are already selected, and an hash `%available-numbers` of the remaing available numbers with a boolean to indicate if the number has already been used.
The the loop is quite simple: for every ring select the value already set, compute which number is required to get 11 and then see if such value is available in the `%available-numbers**.

### Update 2020-01-23

While [reading a blog post about this task](http://blogs.perl.org/users/laurent_r/2020/01/perl-weekly-challenge-43-olympic-rings-and-self-descripting-numbers.html){:target="_blank"}, I realized that I had misinterpreted: *the Black ring is always empty and moreover, some rings has cross-dependencies*.
It appears the task has changed in the middle of the challenge, according to the blog post, however it now has dependencies between rings. Therefore I re-implemented the solution to solve the two-elements rings first, the all the non-black ones, and finally the black one using the numbers of the other rings.
It looks like:

```perl
my @solving-order = <Blue Red Yellow Green Black>;


for @solving-order -> $color {

    # first try to solve the 2-elements rings
    if ( %rings{ $color }.elems == 2 ) {
        my $wanted = 11 - %rings{ $color }[ 0 ];
        if ( %available-numbers{ $wanted } ) {
            %rings{ $color }[ 1 ] = $wanted;
            %available-numbers{ $wanted } = False;
        }
    }
    elsif ( $color !~~ /Black/ ) {
        %rings{ $color }[ 2 ] = %rings{ %rings{ $color }[ 2 ] }[ 1 ];
        my $wanted = 11 - %rings{ $color }.grep( * ~~ Int ).sum;
        %rings{ $color }[ 1 ] = $wanted;
        %rings<Black>.push: $wanted;
    }
}


my $wanted = 11 - %rings<Black>.grep( * ~~ Int ).sum;
%rings<Black>.push( $wanted ) if %available-numbers{ $wanted };

```

Since the rings are walked in a solving order that keeps the black ring as last, it is possible to solve every ring. In the end, the black ring will keep a `Nil` value, so it is quite simple to compute a number that provides the final sum of 11.

## PWC 43 - Task 2

The task 2 was a lot more complicated to me to both understand and implement. The request was to create a *self descriptive number generator*. Essentially, I thought that the problem was to be implemented in two parts:
1) create a validatin function to indicate if a given number was self-descriptive;
2) loop over the generation of every possible number.

[My solution](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/perl6/weekly-challenge/pwc_43_2.p6){:target="_blank"} therefore included the following validatin function:

```perl6
sub validate( Int $number, Int $base ){
    my @digits = $number.Str.split( '', :skip-empty );

    # the number must have the same length
    return False if @digits.elems != $base;

    # the number must have at least one zero!
    return False if ! @digits.grep: 0;

    my $ok = True;
    for @digits {
        my $digit    = $_;
        my $position = $++;
        $ok &&= @digits.grep( * == $position ).elems == $digit;
        return False if ! $ok;
    }

    return True;
}
```

There a couple of *quick exits* in the case the number is not appropriate, for example has not the right length or does not have any zero digit. The validation core is done looping thru all the digits and looking for the number of occurencies of the position of the digit across the string.
<br/>
<br/>
After that, I decided to loop the above function into a number generation, but, uhm...it was really slow. With slow, I mean that the first number in base 10 is `6210001000` it takes a looooong time to get something useful.
Therefore I tried to implement it as a parallel loop, assigning blocks of `10000000000` values to each, but my computer was suffering too much.
Last I found a quick alghoritm to generate them on bases greater than 5, so I applied it (keeping the alghoritm parallel) to get a very quick response. On the other hand, generation of bases lower than 5 was simple enough to make it generating all the numbers. In any case, the program does validate any number before providing it as a solution:

```perl6
sub MAIN( :$base? where { (10,4,5,7,8,9,11,12,16,36).grep: $base } = 10 ){
    say "Starting generation for base $base";

    # I need to do something for other bases!
    die "Not implemented!" if $base > 10;

    if ( $base <= 5 ) {
        my $end = ( $base - 1 ).Str x $base;
        my @nums = grep { validate $_.Int, $base }, 0 .. $end.Int;
        "\nWhat did we generate?\n".say;
        for @nums {
            .say
        }
    }
    else {
        # if the base is greater than 5 do a parallel scan

        my @tasks;
        for ( $base - 4 ) .. ( $base - 1 ) {
            @tasks.push:  Promise.start( {
                 # see <https://medium.com/@divyangrpatel/self-descriptive-number-best-algorithm-95b281e6de05>
                 my $start = '%d21%s1000'.sprintf: $_, '0' x ( $base - 4 - 3 ).Int;
                 my $end   = '%d21%s1000'.sprintf: $_, ( $base - 1 ) x ( $base - 4 - 3 ).Int;
                 say "\tBatch between $start and $end";

                my @nums = grep { validate $_.Int, $base }, $start .. $end;
             } );
        }

        await @tasks;
        "\nWhat did we generate?\n".say;
        for @tasks {
            .result.say if .result;
        }

    }

    say "\nBye bye!";
}
```

Note that the generation is not completed in an available base, but Perl 6 makes it simple enough to be implemented.
