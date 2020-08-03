---
layout: post
title:  "Perl Weekly Challenge 72: trailing zeros and line filter"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 72: trailng zeros and line filter

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 72](https://perlweeklychallenge.org/blog/perl-weekly-challenge-072/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The general situation about the COVID is pretty much the same as the last week. It is still not clear to me what will happen to schools, and I really hope my kid will get the chance to return to school in a *decent* way.

### Olivia
She is trying to make some trouble again, and this is somewhat comfortating. However, she cannot jump nothing higher than a chair.
<br/>
I've kept her in the garden for a few hours, but tied with the dog tie. At least, she is taking some fresh air and getting to know the external world again.


### Eyes

I'm running out of medicines at in a fast time. Besides, no great news, I need to wait for the surgery.

<a name="task1"></a>
## PWC 72 - Task 1

The first task was quite easy: counting the trailing zeros in a factorial of a number.
<br/>
My *five-minutes* solution is the following:

<br/>
<br/>
```perl6
sub MAIN( Int $N where { 0 < $N <= 10 } ) {
    my $factorial = [*] 1 .. $N;
    my $zeros = $factorial ~~ / 0+ $ /;
    "$N ! = $factorial and has { $zeros.chars } trailing zeros".say;
}
```
<br/>
<br/>

Computing the factorial is simple, thanks to the reduction `[*]` operator. Then I applied a regular expression to match any trailing zero, and then ask to the `Match` object how many `chars` the match find out.



<a name="task2"></a>
## PWC 72 - Task 2

The second task was the implementation of something similar to the `head(1)` command: filtering lines within a range.
<br/>
I decided to use a counter to find the first line:

<br/>
<br/>
```perl6
sub MAIN( Str $file-name,
          Int $A where { $A > 0 },
          Int $B where { $B >= $A } ) {
    my $line-counter = 0;
    for $file-name.IO.lines -> $line {
        $line.say if ( $A <= ++$line-counter <= $B );
    }
}
```
<br/>
<br/>

For every `$line` I print it only if the `$line-counter` is within the defined range of lines.
<br/>
There is a chance to get a smarter approach, I thought about playing with `nl-in`, but it does suffice to remember that `IO.lines` is *lazy* and you can store it in an array to get the lines you need:

<br/>
<br/>
```perl6
my @lines = $file-name.IO.lines;
@lines[ $A .. $B ].join( "\n" ).say;
```


