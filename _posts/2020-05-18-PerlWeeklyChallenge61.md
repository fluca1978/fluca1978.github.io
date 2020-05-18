---
layout: post
title:  "Perl Weekly Challenge 61: IPv4 addresses and multiplications"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 61: IPv4 addresses and multiplications

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 61](https://perlweeklychallenge.org/blog/perl-weekly-challenge-061/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation in Italy is ugly and to some extent desperate: it has been 10+ weeks since I'm closed into my house with my son and my wife, we cannot move and cannot go outside. Probably the soldiers will become to monitor the streets very soon this week.
<br/>
<br/>
I've visited my mum twice, but going there is not as simple as it could seem.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


### Olivia
Unluckily, our cat Olivia had a car accident, did undergo a surgery to fix with screws and a fixed external part on her hips.
<br/>
She is at home right now, but not able to move and needs help for feeding.
<br/>
My and my wife are quite sad, because the process is going to be very long and we suffer in not having her jumping around the house.


<a name="task1"></a>
## PWC 61 - Task 1

The first task was about trying the sequence of adiacent numbers in a list that produced the max multiply.
<br/>
I decided to do it with a brute force approach, as often I do: a double nested loop that scans the list and, for every number, finds out the multiplication of the following numbers and tests if the multiplication is higher or not the previous computed one.

```perl6
loop ( my $i = 0; $i < @input.elems; $i++ ){
    for $i ^..^ @input.elems {
         @output = @input[ $i .. $_ ] if ( ! @output || ( [*] @input[ $i .. $_ ] ) > ( [*] @output ) );
    }
}
```

I use the reduction operator, in this case `[*]` that computes the product of the `@output` list. The list is assigned to the current slice of the `@input` array if the product is greated than that computed on `@output` itself.
<br/>
Of course, this does not provide a good performance approach since it would be simpler to cache the current product result, instead of having to recompute it at every iteration, but the code looks a little *smarter* this way.


<a name="task2"></a>
## PWC 61 - Task 2


The second task was about translating a number into an IPv4 address. I remember I read a very clever example with validation into a regexp in the brian's book on Perl 6, but right now does not come into my mind. So my approach was to use the excellent `exhaustive` adverb to regular expressions to get all possible groups of four digits, where each group must be no longer than three digits.
<br/>
Then I convert all the group into integers and store into the `@ip` array, and then I print the array if the number of elements of the array that are within the `0` and `255` range of values is equal to the number of elements of the array (that means, are all good).

```perl6
 for $string ~~ m:ex/ ^ ( <[0..9]> ** 1..3 ) ** 4  $/ {
    my @ip = $_[0].map( *.Int );
    say @ip.join( '.' ) if @ip.grep( 0 <= * <= 255).elems == @ip.elems;
}
```
