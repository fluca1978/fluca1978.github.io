---
layout: post
title:  "Perl Weekly Challenge 106: quick and easy!"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 106: quick and easy!

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 106](https://perlweeklychallenge.org/blog/perl-weekly-challenge-106/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)





<a name="task1"></a>
## PWC 106 - Task 1

The first task was quite easy, even if I did not found a solution more *comfortable* than iterating on a `for` loop. The idea was to sort an array of integers, and get the difference between sequentially adiacent numbers.


### The Raku-ish solution

Thanks to some help, I got to know that `.rotor` can have a *step* argument that can instrument the system to go back, therefore the shortest solution I can come up is:

<br/>
<br/>
```raku
sub MAIN( *@N where { @N.elems == @N.grep( * ~~ Int ).elems } ) {
   @N.sort.rotor( 2 => -1 ).map( { $_[ 1 ] - $_[ 0 ] } ).max.say;
}
```
<br/>
<br/>

That reads as:
- sort the `@N` array;
- extract batchs of `2` elements, and on each batch go back of one position (i.e., `-1`);
- `map` the resulting sub-array to its difference;
- compute the `max` of the resulting array;
- print it out with `say`.
<br/>
Nice!


### Traditional solutions

Without the trick about `rotor`, the other ways to solve the problem did involved looping on the array.
That is:

<br/>
<br/>
```raku
sub MAIN( *@N where { @N.elems == @N.grep( * ~~ Int ).elems } ) {
    my @M = 0;
    my @n = @N.sort;
    @M.push: @n[ $_ + 1 ] - @n[ $_ ]  for 0 ..^ @n.elems - 1;
    say @M.max;
}
```
<br/>
<br/>
The idea is:
- sort the array into `@n`;
- iterate on the sorted array and then compute the difference between a couple of elements, this does not require any *absolute* value since the array is sorted;
- push the result into the `@M` array;
- print out the `max` value of the `@M` array.
<br/>
<br/>

This solution works fine, but it does consume memory to store all intermediate results, and of course there is the possibility to use a single scalar value instead of the `@M` array:
<br/>
<br/>
```raku
 my $max = 0;
 $max = @n[ $_ + 1 ] - @n[ $_ ] > $max
                   ?? @n[ $_ + 1 ] - @n[ $_ ]
                   !! $max  for 0 ..^ @n.elems - 1;
 say $max;
```
<br/>
<br/>

The trick is the same, but this time `$max` stores only the difference when it is greater than the previously computed difference.



<a name="task2"></a>
## PWC 106 - Task 2

The second task was easy too, because Raku already provides the very same example in the documentation. The task was about printing out a decimal number coming from a rational one with the periodic part in parenthesis.
<br/>
This is simple enough because `Rat` provides the `base-repeat` method that gives an array where the result is split into the non-repeating part and the repeating one.
The task therefore is:

<br/>
<br/>
```raku
sub MAIN( Int $D, Int $N where { $N != 0 } ) {
    my @values = ( $D / $N ).base-repeating( 10 );

    print @values[ 0 ];
    print '(%s)'.sprintf: @values[ 1 ] if ( @values[ 1 ] );
    put '';
}
```
<br/>
<br/>

As you can see, there is much more code involved in the printing phase that in the computing one.
<br/>
To shorten the printing phase, it is possible to do something like the following:

<br/>
<br/>
```raku
sub MAIN( Int $D, Int $N where { $N != 0 } ) {
    my @values = ( $D / $N ).base-repeating( 10 );

    @values[ 0 ].say and exit if ( ! @values[  1 ] );
    '%s(%s)'.sprintf( @values ).say;
}
```
<br/>
<br/>
Therefore, the `printf` approach is used only if there is the repeting part, otherwise a *simple* `say` is used and the program is terminated. Please note the usage of low priority `and` before the `exit`, otherwise the program will terminate without printing anything.



