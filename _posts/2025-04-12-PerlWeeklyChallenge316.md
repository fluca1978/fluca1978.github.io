---
layout: post
title:  "Perl Weekly Challenge 316: coming back from PostgreSQL OpenDay"
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

# Perl Weekly Challenge 316: coming back from PostgreSQL OpenDay

This post presents my solutions to the [Perl Weekly Challenge 316](https://perlweeklychallenge.org/blog/perl-weekly-challenge-316/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 316 - Task 1 - Raku](#task1)
- [PWC 316 - Task 2 - Raku](#task2)




# Raku Implementations

<a name="task1"></a>
## PWC 316 - Task 1 - Raku Implementation

The first task was about searching for in a list of words, to see if the last letter of a word is the first one of the next word in the list.



<br/>
<br/>
```raku
sub MAIN( *@words where { @words.elems > 0 } ) {

    for 0 ..^ @words.elems - 1 {
		my $ending = @words[ $_ ].comb[ * - 1 ].fc;
		my $beginning = @words[ $_ + 1 ].comb[ 0 ].fc;
		'False'.say and exit unless ( $beginning ~~ $ending );
    }

    'True'.say;
}

```
<br/>
<br/>

The idea is simple: I iterate over the list of words and extract the `$ending` letter of the current word and the `$beginning` letter of the next word. If the two letters do not match, I end the program. If the loop ends, then all words satisfy the constraint, so the program was sucessful.



<a name="task2"></a>
## PWC 316 - Task 2 - Raku Implementation

The second task was to find out if a given word can be a subsequence of another one, i.e., if all the letters of the first word are contained in the second word in the same order, even if interleaved with other chars.

<br/>
<br/>
```raku
sub MAIN( Str $original, Str $subsequence ) {
    my $index = 0;
    for $original.lc.comb  {
	'False'.say and exit unless ( $subsequence.comb[ $index .. * ].grep( $_ ) );
	$index = $subsequence.comb[ $index .. * ].grep( $_,  :k )[ 0 ];
    }

    'True'.say;

}
```
<br/>
<br/>

The solution is simple: I iterate over the letters of the first word and see if I can find it in a sub-array of the second word.

