---
layout: post
title:  "Perl Weekly Challenge 346: really not inspired!"
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

# Perl Weekly Challenge 346: really not inspired!

This post presents my solutions to the [Perl Weekly Challenge 346](https://perlweeklychallenge.org/blog/perl-weekly-challenge-346/){:target="_blank"}.

<br/>
This week, I solved the following tasks:

<br/>
- [PWC 346 - Task 1 - Raku](#task1)

# Raku Implementations

<a name="task1"></a>
## PWC 346 - Task 1 - Raku Implementation

The first task was about counting the largest set of balanced parentheses in a paren-only string.



<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    my @chars = $string.comb;
    my %positions;
    my $level = 0;
    for 0 ..^ @chars.elems {
		if ( @chars[ $_ ] ~~ '(' ) {
		    %positions{ $level }<start> = $_;
		    $level++;
		}
		elsif ( @chars[ $_ ] ~~ ')' ) {
		    $level--;
		    %positions{ $level }<end> = $_;

		}
    }

    # now get the max pair
    my $max-length = 0;
    for %positions.keys {
		my $current = %positions{ $_ }<end> - %positions{ $_ }<start>;
		$max-length = $current if ( $current > $max-length );
    }

    $max-length.say;
}

```
<br/>
<br/>


I solved this by keeping track, into `%positions` of the level of nesting of parens, and the `start` end `and` chars for the opening and closing parentheses. Then I search the longest one match.
