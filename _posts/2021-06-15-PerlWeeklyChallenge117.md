---
layout: post
title:  "Perl Weekly Challenge 117: quick and dirty"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 117: quick and dirty

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 117](https://perlweeklychallenge.org/blog/perl-weekly-challenge-117/){:target="_blank"}.

<br/>
My official solutions are available [on the GitHub repository](https://github.com/fluca1978/perlweeklychallenge-club/tree/master/challenge-117/luca-ferrari){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<a name="task1"></a>
## PWC 117 - Task 1

The first task was about finding outmissing numbered lines in a file. Something that can be very helpful if you are programming in Basic!

<br/>
<br/>
<br/>
<br/>
```raku
sub MAIN( Str $file-name = 'test.txt' ) {
    my %lines = $file-name.IO.lines.map: { my ( $number, $content ) = .split( ',' );
                                           $number.Int => $content.trim;
                                         }

    say "Missing line $_ !" if %lines{ $_ }:!exists
               for 1 .. %lines.keys.sort(  *.Int <=> *.Int )[ * - 1 ];
    
}

```
<br/>
<br/>

I do read all the lines, and place them into an hash where the number is the key and the content is the rest of the line.
Thenm I iterate on all numbers of lines, in particular from 1 to the max value found, and I search for that key in the hash. If the key does not exist, the line is missing.

<a name="task2"></a>
## PWC 117 - Task 2

This is something I'm not sure I've done right, but essentially is the moves you can get from the root of a triangle structure to the rightmost bottom.
I coded the moves you can do, and then I expand the moves depending on the deep of the triangle. I'm not sure this is going to work, without any missing move, for very large triangles:

<br/>
<br/>
```raku
sub MAIN( Int $n ) {

    my @directions.push:
    'R' x $n 
    ,  'LH' x $n 
    ,  'L' x $n ~  'H' x $n 
    ,  'LH' x ($n -1) ~ 'R' x ($n - 1 )
    ,  'R' x ($n - 1) ~ 'LH' x ($n - 1 )
    ,  'R' x ($n -1) ~ 'LH' x ($n - 1);

    @directions.unique.say;
}
```
<br/>
<br/>
