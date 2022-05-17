---
layout: post
title:  "Perl Weekly Challenge 165: SVG"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 165: SVG

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 165](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0165/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 165 - Task 1

This task was about to plot a set of points and lines inserted from the standard input.
I used the `SVG` module, that in turns is a wrapper to produce an XML document that represents an SVG image.



<br/>
<br/>
```raku
use SVG;
sub MAIN( Str $filename = 'task1.svg' ) {
    my ( @points, @lines );

    for $*IN.lines() -> $line {
        my @elements = $line.split(',').map( *.trim );
        next if @elements.elems !%% 2 && @elements.elems !%% 4;

        if @elements.elems == 2 {
            # a point
            my $point = circle =>  [ cx => @elements[ 0 ].Num,
                                     cy => @elements[ 1 ].Num,
                                     r => 5,
                                     fill => 'blue' ];
            @points.push: $point;
        }
        else {
            # a line
            my $line = line => [ x1 => @elements[ 0 ].Num,
                                 y1 => @elements[ 1 ].Num,
                                 x2 => @elements[ 3 ].Num,
                                 y2 => @elements[ 3 ].Num,
                                 stroke => 'magenta' ];
            @lines.push: $line;
        }

    }

    $filename.IO.spurt( SVG.serialize:
            svg => [ width => 500, height => 500, |@points, |@lines ] );
}

 ```
<br/>
<br/>

The idea is quite simple. First of all I split the input into number pairs and if the user has inserted something that is not a number nor a correct pair, I skip the input line.
Then I check: if there re only two numbers, it is a point, otherwise it must be a line.
Depending on that, I create an hash containing the coordinates with names as they have to appear in the SVG graphic (here I took some examples before I was ready to plo!).
Last I ask `SVG` to *plot* the result in a `500x500` canvas. The resulting XML is `spurt`ed to a file, so that the image is effectively stored on the hard drive.


<a name="task2"></a>
## PWC 165 - Task 2

A task to plot the nearest line for a set of points using the previous program.

<br/>
<br/>
```raku
sub MAIN() {

    my @input-points =
                <333,129  39,189 140,156 292,134 393,52  160,166 362,122  13,193
                 341,104 320,113 109,177 203,152 343,100 225,110  23,186 282,102
                 284,98  205,133 297,114 292,126 339,112 327,79  253,136  61,169
                 128,176 346,72  316,103 124,162  65,181 159,137 212,116 337,86
                 215,136 153,137 390,104 100,180  76,188  77,181  69,195  92,186
                 275,96  250,147  34,174 213,134 186,129 189,154 361,82  363,89 >;

    # "decompose" the input into two elements arrays
    my @points = @input-points.split( / \s+ / ).split( ',' ).split( / \s+ / ).map( *.Int ).rotor: 2;


    # compute all the parts
    my ( $m, $x, $y, $xy, $xx ) = 0,0,0,0,0;
    $x  = [+] @points.map( { $_[0] } );
    $y  = [+] @points.map( { $_[1] } );
    $xy = [+] @points.map( { $_[0] * $_[1] } );
    $xx = [+] @points.map( { $_[0] ** 2 } );
    $m  = ( @points.elems * $xy - $x * $y ) / ( @points.elems * $xx - $x ** 2 );

    my $b = 0;
    $b = ( $y - $m * $x ) / @points.elems;

    # compute the line start and end point
    my @line;
    @line.push: $_, $m * $_ + $b for 0,100;


    # now I need to graph
    my $task1 = run "raku", <raku/ch-1.p6 task2.svg>, :in, :err;
    $task1.in.say: $_.join( ',' ) for @points;
    $task1.in.say: @line.join( ',' );
    $task1.in.close;
    $task1.err.slurp.say;
}

```
<br/>
<br/>

First of all, `@points` is an array of couple of points, so that the previous program can use it as its input. For gaining such couples, there is a little mangling to be done, with particular regard to spaces.
<br/>
Then I compute the `y = mx + b` formula, or better `$m` and `$b` and I compute the starting and ending points using `x` with `0` and `100`.
With all that set, I can now invoke the previous program using Raku process faciliies and passing all the points and then the ling as `:in` standard input.
