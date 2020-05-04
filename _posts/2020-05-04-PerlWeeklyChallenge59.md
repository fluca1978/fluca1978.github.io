---
layout: post
title:  "Perl Weekly Challenge 59: bits and arrays"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 59: bits and arrays

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 59](https://perlweeklychallenge.org/blog/perl-weekly-challenge-059/){:target="_blank"}.
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
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 59 - Task 1

The first task was too me simple, that makes me think I have not understood it very well!
<br/>
The task required you to split a list of numbers so that on he left of a number you have all lesser than digits, followed by greater or equal digits keeping the same ordering within the two partitions.
<br/>
My personal solution was as simple as:

```perl6
sub MAIN( Int:D :$k = 3 ) {

    my @L = 1, 4, 3, 2, 5, 2 ;
    my @l = | @L.grep( * < $k ),  | @L.grep( * >= $k );
    say "Index $k makes { @L } to become { @l }";
}
```

<br/>
In short: the first partition is made by all digits lesser than my `$k` number, then all the remaining numbers. Since the `.grep` preserves the order of the lists, the combined list (flattened by `|`) keeps the order too.
In other words:

```shell
% raku ch-1.p6 --k=4
Index 4 makes 1 4 3 2 5 2 to become 1 3 2 2 4 5
```


<a name="task2"></a>
## PWC 59 - Task 2


This task was a little longer than the first one: you have to built a function `f(a,b)` that reports the number of bits different between the two representations of the arguments. That's quite easy, but the two digits could end up having different bit lengths, so padding is required.
<br/>
My implementation to such function is:

```perl6
sub f( Int:D $a, Int:D $b ) {
    my $different-bits = 0;

    my @a-bits = $a.base( 2 ).Str.comb.reverse;
    my @b-bits = $b.base( 2 ).Str.comb.reverse;

    # find the longest number
    my $max-length = max( @a-bits.elems, @b-bits.elems );
    # do the padding with zeros (to the end, the arra)
    @a-bits.push: 0 for 0 .. ( $max-length - @a-bits.elems  );
    @b-bits.push: 0 for 0 .. ( $max-length - @b-bits.elems );


    # compute the difference
    for 0 ..^ @a-bits.elems {
        $different-bits += 1 if ( @a-bits[ $_ ] != @b-bits[ $_ ] );
    }

    $different-bits;
}

```


<br/>
First of all, I ask `raku` to rewrite the digits in a binary format by means of `.base(2)`, then I convert to a string and split into an array of single bits, tht I then `.reverse` to get the less significative bit as element zero of the array.
Then it is simple to pad the arrays of bits: the shortest array gets a few `0` pushed to its end (that represent the most significative bits). I don't care to know what number has the shortest bnary representation, so I pad both the arrays with the xcept that one padding will not execute.
<br/>
Then it does suffice to walk the arrays, that now have the very same length, and count how many bits (array elements) are different.
<br/>
<br/>
Having done so, the task asked to sum the number of different bits among couple of arguments, as easy as:


```perl6
my $sum = 0;
for 0 ..^ @*ARGS.elems  -> $first {
    for $first + 1 ..^ @*ARGS.elems  -> $second {
        $sum += f( @*ARGS[ $first ].Int, @*ARGS[ $second ].Int );
    }
}
say "Sum is $sum";
```

<br/>
A nested `for` to make all the available couples of digits and sum everything.
