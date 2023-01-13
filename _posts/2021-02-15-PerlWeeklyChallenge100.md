---
layout: post
title:  "Perl Weekly Challenge 100: times and triangles"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 100: times and triangles

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 100](https://perlweeklychallenge.org/blog/perl-weekly-challenge-100/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




# My eyes...

I'm a little stressed in this period because I'm having hard times at work, and this is of course not so good on my eyes side.
<br/>
However, I started to move a little more and, despite the difficulties, I'm gaining more independence. This could sound trivial, but after more than four months of *almost blindness*, it is a huge improvement in my life!

# And PWC hitted 100!

*This is the **one hundred** Perl Weekly Challenge**, and I'm glad this project do exist and puzzles developers with sweet and short tasks to help they improve their skills.
<br/>
**A big thank you Mohammed** and everyone that, with patches, reviews and blog post is contributing to the project.

<a name="task1"></a>
## PWC 100 - Task 1
The first task was a kind of *12-24 hour converter*: the user can specify one or two strings to specify an hour, and the script has to be smart enough to understand the time format and convert it into another format.

<br/>
I was unable to write a *one-liner*, so here it is my solution:

<br/>
<br/>
```raku
sub MAIN( Str $when, Str $phase? = '' ) {
    if $when ~ $phase ~~ / $<hour>=\d+ ':' $<minute>=\d+ $<phase>=( 'am'|'pm' )? / {
        my $hour = $<hour>.Int;
        my $am   = "%s".sprintf( $<phase> && $<phase>.Str ?? $<phase>.Str.lc !! '' );

        given $am {
            when 'am' { $hour += $hour > 12 ?? -12 !! 0; $am = ''; }
            when 'pm' { $hour += $hour < 12 ??  12 !! 0; $am = ''; }
            default   { $am = $hour < 13 ?? 'am' !! 'pm';
                        $hour += $hour < 13 ?? 0   !! -12; }
        }

        say "%02d:%02d %s".sprintf( $hour , $<minute>, $am ).trim;
    }
}

```
<br/>

The idea is as follows:
- I concatenate the two possible input strings, with the latest one being an empty string by default to prevent `Nil` concatenation. If the string obtained matches a regular expression, I capture the single parts;
- compute the `$hour` value as integer, since this is the only value that is going to be chaned;
- compute an `$am` flag that contains the lower case final part, if any;
- depending on the value of `$am` I change the value of `$hour` and in the case there is no `$am` I set the value of `$am` to a string;
- last, I print the whole string with `$am` possibly set to null, so I `trim` the string before printing it out.



<a name="task2"></a>
## PWC 100 - Task 2

The second task was about finding the *minimum path* in a triangle of numbers.
<br/>
This results as a one line where I `sum` the `min` of every level of the array of array:


<br/>
<br/>
```raku
sub MAIN() {
    my @triangle = [ [1], [2,4], [6,4,9], [5,1,7,2] ];
    my $sum += .min for @triangle;
    say $sum;
}
```
<br/>
<br/>

