---
layout: post
title:  "Perl Weekly Challenge 109: Choowa Numbers and Sums"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 109: Choowa Numbers and Sums

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 109](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0109/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


## Eyes

Last monday I got my **second sub-cataract laser treatment** against my severally damaged left eyes.
<br/>
There is little hope for such an eye, that so far is going to see less than `1/10`.
<br/>
But a little more light is better than less, right?


<a name="task1"></a>
## PWC 109 - Task 1

The first task was about producing *Choowa* numbers, that are in short the summatory of all divisors for a specific order.
<br/>
I decided to implement a `choowa` function that accepts the `$n` order and computes all the *divisors* of `$n`: that is quite simple since it means searching for any number that provides no reminder on a division. I corced them and used the `[+]` reduction operator to return the sum of those numbers. Please note that there is the special case where `$n` is `1` or `0`, and in such case *Choowa* says the divisors should not be took into account.
<br/>
In the `MAIN` I iterate on the order up to `$limit` and invoke the function, than coerce the result into an array and print it. Therefore, the whole program is:

<br/>
<br/>
```raku
sub choowla( Int $n where { $n > 0 } ) {
    return 0 if $n < 2;
    return [+] ( $_ if $n %% $_ for 2 .. $n / 2 );
}

sub MAIN( Int $limit = 20 ) {
    @( choowla( $_ ) for 1 .. $limit ).join( ',' ).say;
}

```
<br/>
<br/>


<a name="task2"></a>
## PWC 109 - Task 2
The second task was about finding out a number scheme in which different sums provide the same result.
<br/>
In particular, given numbers called from `a` to `g`, the following sums must have the equal values:
- `a` + `b`;
- `b` + `c` + `d`;
- `d` + `e` + `f`;
- `f` + `g`.
<br/>
I decided to implement an hash named `%squares` that contain every letter as a key and the value assigned to the letter as the number to sum. Then, I push all the sums into an array named `@sums`, and this can be done with the `[+]` reduction operator and the slicing of the hash itself.
<br/>
Therefore `[+] %squares{ 'b' .. 'd' };` is the sum of `b` + `c` + `d`.
<br/>
Last, I check if the first sum in the array `@sums` is equal to **all** the remaining sums in the array, and this is done with the `all` special keyword for junctions and the array slicing.
<br/>
If the sum condition is met, I push the current `%squares` configuration hash to a `@solutions` array, so that I can print it at the end of the script.
<br/>
The script therefore is:


<br/>
<br/>
```raku
multi sub MAIN(  *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems && @nums.elems == 7 } ) {
    my @letters = 'a' .. 'g';
    my @solutions;

    # permutate all numbers to find out the correct sum
    for @nums.permutations -> @current {
        # build an hash to help visualize the answer
        my %squares;
        %squares{ @letters[ $_ ] } = @current[ $_ ] for 0 ..^ @letters.elems;

        # compute the sums
        my @sums;
        @sums.push: [+] %squares{ 'a' .. 'b' };
        @sums.push: [+] %squares{ 'b' .. 'd' };
        @sums.push: [+] %squares{ 'd' .. 'f' };
        @sums.push: [+] %squares{ 'f' .. 'g' };


        # if the first sum is equal to all the others, push this solution
        @solutions.push: %squares if @sums[ 0 ] == all @sums[ 1 .. * ];
    }



    say $_ for @solutions;

}


multi sub MAIN() {
    MAIN( <1 2 3 4 5 6 7> );
}
```
<br/>
<br/>

Note the usage of a `multi MAIN` to allow the user to run the script without having to specify any value.
