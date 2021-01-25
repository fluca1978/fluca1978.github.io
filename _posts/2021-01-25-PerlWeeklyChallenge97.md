---
layout: post
title:  "Perl Weekly Challenge 97: flipping and swapping"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 97: flipping and swapping

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 97](https://perlweeklychallenge.org/blog/perl-weekly-challenge-097/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




# My eyes...

I'm waiting and hoping for another consultancy, the next week, by another specialied center.

<a name="task1"></a>
## PWC 97 - Task 1

The first task was about to implement a Caesar Cipher, that is to translated every single letter in a phrase with a *swapped* one in the alphabet.
<br/>
The idea was to have a program that with a line like the following could do the trick:

<br/>
<br/>
```raku
print %cipher{ $_ }:exists ?? %cipher{ $_ } !! $_ for $S.comb;
```
<br/>
<br/>

where `%cipher` is a *shifted* alphabet where every single letter of an original alphabet is transposed to a different letter. For example, assuming you are ciphing with `3` the letter `A` is translated to `X`, `B` to `Y` and so on. In the above, the assumption is that the `%cipher` hash contains the key as the unencrypted letter and the value as the encrypted one, such as `%cipher{ 'A' } = 'X'`.
<br/>
Now, the main part is to build the `%cipher` hash. In the beginning I took a quite simple and straightforward approach like:

<br/>
<br/>
```raku
my $index = @alphabet.elems - $N;
for @alphabet {
    %cipher{ $_ } = @alphabet[ $index ];
    $index = $index + 1 < @alphabet.elems ?? $index + 1 !! 0 ;
}
```
<br/>
<br/>

Having `@alphabet` being the standard alphabet, and `$N` the cypher offset, every element in the `%cipher` hash is keyed to the current alphabet letter and the *modulo* index of the alphabet itself.
<br/>
Then, I remembered about the `rotate` function in an array, that does exactly what the above code does. Therefore, the whole program becomes:


<br/>
<br/>
```raku
sub MAIN( Str $S = "THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG",
          Int $N where { $N > 0 && $N < ( 'A' .. 'Z' ).elems } = 3 ) {
    my @alphabet = 'A' .. 'Z';
    my %cipher;

    %cipher{ @alphabet[ $_ ] } = @alphabet.rotate( $N * -1  )[ $_ ] for ^@alphabet.elems;

    say "Encoding $S";
    print %cipher{ $_ }:exists ?? %cipher{ $_ } !! $_ for $S.comb;
    say "\ndone";
}
```
<br/>
<br/>

With a single line I can initialize the `%cipher` hash using the `rotate` method and extracing the current element corresponding to a letter. Of course, invoking `@alphabet.rotate( $N * -1  )[ $_ ]` on every single initialization is expensive, therefore materializing the rotation could improve the script performances.



<a name="task2"></a>
## PWC 97 - Task 2

The second task was about computing the number of *flips* required to obtain a binary string, previously splitted. The main idea, as far as I understand from the examples, is to split a binary string into chuks, then take the first chunk and see how many bit changes are required to produce all subsequent chunks.
<br/>
I implemented it as follows:

<br/>
<br/>
```raku
sub MAIN( Str $B = "101100101",
          Int $S = 3 ) {

    my @splits = $B.comb: $S.Int;
    my $flips = 0;

    my ( $a, $b ) = @splits[ 0 ], Nil;

    for 1 ..^ @splits.elems  {
        $b = @splits[ $_ ];

        # find out how many chars are different
        $flips++ if ( $a.comb[ $_ ] != $b.comb[ $_ ] ) for ^@$a.comb.elems;
    }

    say $flips;
}
```
<br/>
<br/>

The idea is to create first the array of chunks, named `@splits`. Then `$a` is the first chunk and `$b` is the next chunk to analyze. For every character that is not the same (on the same position) between the two chunks, I increment the `$flips` counter.
