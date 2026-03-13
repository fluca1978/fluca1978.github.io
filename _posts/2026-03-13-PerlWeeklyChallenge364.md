---
layout: post
title:  "Perl Weekly Challenge 364: substituting strings!"
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

# Perl Weekly Challenge 364: substituting strings!

This post presents my solutions to the [Perl Weekly Challenge 364](https://perlweeklychallenge.org/blog/perl-weekly-challenge-364/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 364 - Task 1 - Raku](#task1)
- [PWC 364 - Task 2 - Raku](#task2)


# Raku Implementations

<a name="task1"></a>
## PWC 364 - Task 1 - Raku Implementation

Given a string that contains numbers and `#` translate it into a letters-only string.
The first part of the alphabet is mapped with numbers ranging from `1` to `9`, while the second part with numbers followed by a `#`.



<br/>
<br/>
```raku
sub MAIN( Str $string where { $string ~~ / ^ <[0 .. 9 #]>+ $ / }) {
    my $result = $string;
    my @alphabet = 'a' .. 'z';
    $result .= subst( / ( <[1..2]> <[0..6]> ) '#'/,
		      { @alphabet[ $/[0].Int - 1 ] } ,
						     :g );

    $result .= subst( / (<[1 .. 9 ]>) /, { @alphabet[ $/[ 0 ].Int - 1 ] }, :g );
    say $result;
}

```
<br/>
<br/>

First of all, I remove all the double digits values, so I substitute all the double digits with letters. This gives me a string amde by letters and single digits, that can then be substituted in the second run.



<a name="task2"></a>
## PWC 364 - Task 2 - Raku Implementation

Given a strange *goal* string, convert it.

<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    my $result = $string;
    $result .= subst( / '()' /, 'o', :g );
    $result .= subst( / '(al)' /, 'al', :g );
    say $result;
}

```
<br/>
<br/>

This is a two pass substitution, that can be shrinked into a single one, but for readability sake I kept all the passes explicit.
