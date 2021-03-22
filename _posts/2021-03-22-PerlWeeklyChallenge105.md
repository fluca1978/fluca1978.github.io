---
layout: post
title:  "Perl Weekly Challenge 105: singing roots"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 105: singing roots

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 105](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0105/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)





<a name="task1"></a>
## PWC 105 - Task 1

The first task was about computing the *nth root* of a number. The script takes two integer arguments: the root level and the number of which I need to compute the root.
<br/>
In Raku the task seems enough simple thanks to the `.roots` method that computes all the roots of a number. In particular, the one with no imaginary part (`.roots` computs `Complex` roots) is the one I am looking for. The script therefore reduces to a single line:

<br/>
<br/>
```raku
sub MAIN( Int $N = 5, Int $K = 248832 ) {
    "Computing $N root of $K".say;
    "%.2f".sprintf( .re ).say 
             for $K.roots( $N ).grep: { ! .im.Int };
}
```
<br/>
<br/>
That reads as:
- for every `.roots` of level `$N` of the number `$K`;
- search for (i.e., `grep`) every number that has the imaginery part `.im` zero;
- produce a string with two decimals of the real `.re` part of such number;
- print it with `say`.

<br/>
I think this is a correct solution, even if it is not clear to me if there are other cases where this approach is not correct. Also note that I'm not aware of numbers that can have more than one real root, but to avoid problems I decided to substitute the `for` loop with a `given` that extracts the first one entry from the list:

<br/>
<br/>
```raku
sub MAIN( Int $N = 5, Int $K = 248832 ) {
    "Computing $N root of $K".say;
    "%.2f".sprintf( .re ).say 
             given $K.roots( $N ).grep( { ! .im.Int } )[ 0 ];
}
```
<br/>
<br/>


<a name="task2"></a>
## PWC 105 - Task 2

The second task was about the *name game song*: produce a stanza with a mangled person's name. As far as I understand, the rules are to remove the initial letter from the name depending on its value.
<br/>
What I did was to extract the `$first-char` of the name, and depending on it being a wovel or not, produce a `$mangled-name` that is the same name without its initial char or not.
Then I produce an array of `@special-chars` that contain a single letter or an empty space depending again on the first char of the name.
<br/>
Finally, I produce the stanza with an *heredoc* quoting composing the mangled name and the special char on every line they are required.

<br/>
<br/>
```raku
sub MAIN( Str $name = 'Katie' ) {
    my $first-char = $name.substr( 0, 1 ).lc;
    my $mangled-name = qw< a e i o u >.grep( * ~~ $first-char ) 
                         ?? $name 
                         !! $name.substr( 1, $name.chars ).lc;
    my @special-chars = qw< b f m >.map: 
                         { $_ !~~ $first-char ??  $_  !!  '' };

    say qq:to/SONG/;
    $name, $name bo-{ @special-chars[ 0 ] ~ $mangled-name.lc }
    Bonana-fanna fo-{ @special-chars[ 1 ] ~ $mangled-name.lc }
    Fee fi mo-{ @special-chars[ 2 ] ~  $mangled-name.lc }
    $name !
SONG
}

```
<br/>
<br/>


