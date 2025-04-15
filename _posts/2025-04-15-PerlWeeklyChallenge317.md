---
layout: post
title:  "Perl Weekly Challenge 317: only Raku for now!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 317: only Raku for now!

This post presents my solutions to the [Perl Weekly Challenge 317](https://perlweeklychallenge.org/blog/perl-weekly-challenge-317/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 317 - Task 1 - Raku](#task1)
- [PWC 317 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 317 - Task 1 - Raku Implementation

The first task was to find out if the first letter of a list of words created an acronym passed as argument.



<br/>
<br/>
```raku
sub MAIN( Str $acronym, *@words where { @words.elems == $acronym.chars } ) {
    'True'.say and exit if ( @words.map( *.fc.comb[ 0 ] ).join ~~ $acronym.fc );
    'False'.say;
}

```
<br/>
<br/>

This is quite simple: I extract all the first char of the given list of `@words` and `join` it back to compare with the smart match against the `$acronym`.



<a name="task2"></a>
## PWC 317 - Task 2 - Raku Implementation

The second task was to find out if exchanging any two characters in a word resulted in the other given word.

<br/>
<br/>
```raku
sub MAIN( Str $needle, Str $haystack where { $haystack.chars == $needle.chars } ) {
    # short circuit: the strings are the same
    'True'.say if ( $needle.fc ~~ $haystack.fc );

    my $differences = 0;
    my @needle = $needle.lc.comb;
    my @haystack = $haystack.lc.comb;

    for 0 ..^ @needle.elems {
		$differences++ unless ( @needle[ $_ ] eq @haystack[ $_ ] );
    }

    'True'.say and exit if ( $differences == 2 );
    'False'.say;
}

```
<br/>
<br/>


This is quite simple: I keep a list of characters of the two strings into `@needle` and `@haystack`.
Then I iterate over the first of the two and count how many `$differences` there are, assuming a difference is the fact that the character at the very same position in the other string is not equal.
If the `$differences` are exactly two, the task is succesful.
