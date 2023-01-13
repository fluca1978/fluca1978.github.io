---
layout: post
title:  "Perl Weekly Challenge 152: St. Valentine's sums and triangles"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 152:  St. Valentine's sums and triangles

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 152](https://perlweeklychallenge.org/blog/perl-weekly-challenge-152/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)





<a name="task1"></a>
## PWC 152 - Task 1

The first task was very easy: given a nested array of integers, where each sub-array is a level of a triangle, compute the less expensive path from top to bottom.
<br/>


<br/>
<br/>
```raku
sub MAIN() {
    my @triangle = [ [1], [5,3], [2,3,4], [7,1,0,2], [6,4,5,2,8] ];
    my $sum += $_.min for @triangle;
    $sum.say;
}
 ```
<br/>
<br/>

Essentially, do make the sum of all the mins for every level of the triangle.


<a name="task2"></a>
## PWC 152 - Task 2

Gosh, overlapping rectangles! It took me more time to understand the problem than to implement it.
You are given two rectangles (by their opposite corners), and must compute the *total area* covered by the two, meaning that you need to compute the sum of the areas without double summing the overlapping one.
<br/>
I decided to start it easy: design a `Point` and a `Rectangle` class:

<br/>
<br/>
```raku
class Point {
    has Int $.x;
    has Int $.y;
}


class Rectangle {
    has Point $.corner-left;
    has Point $.corner-right;

    method area() {
        abs( $!corner-right.x - $!corner-left.x )
        * abs( $!corner-right.y - $!corner-left.y );
    }

    method overlapping-area( Rectangle $r ) {
        my $left = Point.new( x => max( $!corner-left.x, $r.corner-left.x ),
                              y => max( $!corner-left.y, $r.corner-left.y )
                            );
        my $right = Point.new( x => min( $!corner-right.x, $r.corner-right.x ),
                               y => min( $!corner-right.y, $r.corner-right.y )
                             );

        my Rectangle $overlapping = Rectangle.new:
                                    corner-left =>  $left,
                                    corner-right => $right;
        return $overlapping.area;

    }
}

```
<br/>
<br/>

The `Rectangle` class has two interesting methods:
- `area` computes the current are of the rectangle;
- `overlapping-area` computes the overlapped area between the current rectangle and an overlapping one.

The last method considres the coordinates of the overlapping rectangles, and computes the area of such rectangle.
<br/>
Therefore, the `MAIN` results in:

<br/>
<br/>

``` raku
sub MAIN(  *@points where { @points.elems == 8 && @points.grep( * ~~ Int ) == @points.elems } ) {
    my @corners;
    @corners.push: Point.new( x => $_[ 0 ].Int, y => $_[ 1 ].Int ) for @points.rotor( 2 );
    my Rectangle $r1 = Rectangle.new: corner-left => @corners[ 0 ], corner-right => @corners[ 1 ];
    my Rectangle $r2 = Rectangle.new: corner-left => @corners[ 2 ], corner-right => @corners[ 3 ];

    "{ $r1.area + $r2.area - $r1.overlapping-area( $r2 ) }".say;
}

```
<br/>
<br/>

First of all, I build up the list of `Point`s out of the incoming list of arguments, then I build the two `Rectangle`s and sum the areas removing the `overlapping-area`.


### What about non-overlapping rectangles?

The above program does not control that the two rectangles are effectively overlapping, thus in the case of non overlapping rectangles, the resulting area is totally wrong because it is subtracted by a piace of area that is not covered at all.
<br/>
Fear not: it is easy to improve the situaion. First of all, I created an `is-overlapping` private method that checks if the two rectangles are overlapping. Then, the `overlapping-area` method is changed to ignore non overlapping rectangles:


<br/>
<br/>

``` raku
class Rectangle {
    has Point $.corner-left;
    has Point $.corner-right;

    method area() {
        abs( $!corner-right.x - $!corner-left.x )
        * abs( $!corner-right.y - $!corner-left.y );
    }


    method !is-overlapping( Rectangle $r ) {
        ( $!corner-left.x <= $r.corner-left.x <= $!corner-right.x
           ||  $!corner-left.x <= $r.corner-righ.x <= $!corner-right.x )
        &&
        ( $!corner-left.y <= $r.corner-left.y <= $!corner-right.y
          ||  $!corner-left.y <= $r.corner-right.y <= $!corner-right.y );

    }

    method overlapping-area( Rectangle $r ) {
        return 0 if ! self!is-overlapping( $r );

        my $left = Point.new( x => max( $!corner-left.x, $r.corner-left.x ),
                              y => max( $!corner-left.y, $r.corner-left.y )
                            );
        my $right = Point.new( x => min( $!corner-right.x, $r.corner-right.x ),
                               y => min( $!corner-right.y, $r.corner-right.y )
                             );

        my Rectangle $overlapping = Rectangle.new:
                                    corner-left =>  $left,
                                    corner-right => $right;
        return $overlapping.area;

    }
}

```
<br/>
<br/>

### Using `map` instead of `push`

Is there room for another little improvement: using `map` to build up the list of corners.

<br/>
<br/>

``` raku
sub MAIN(  *@points where { @points.elems == 8 && @points.grep( * ~~ Int ) == @points.elems } ) {
    my @corners = @points.rotor( 2 ).map: { Point.new( x => $_[ 0 ].Int, y => $_[ 1 ].Int ) };
    my Rectangle $r1 = Rectangle.new: corner-left => @corners[ 0 ], corner-right => @corners[ 1 ];
    my Rectangle $r2 = Rectangle.new: corner-left => @corners[ 2 ], corner-right => @corners[ 3 ];

    "{ $r1.area + $r2.area - $r1.overlapping-area( $r2 ) }".say;
}

```
<br/>
<br/>

The `@corners` array is built on the fly using `map` on every couple of coordinates, and creating the respective `Point`.
<br/>
The same *trick* can be applied to the building of `Rectangle`s, therefore the final `MAIN` results in:


<br/>
<br/>

``` raku
sub MAIN(  *@points where { @points.elems == 8 && @points.grep( * ~~ Int ) == @points.elems } ) {
    my @corners = @points.rotor( 2 ).map: { Point.new( x => $_[ 0 ].Int, y => $_[ 1 ].Int ) };
    my Rectangle ( $r1, $r2 ) = @corners.rotor( 2 ).map: { Rectangle.new(
                                                                 corner-left => $_[ 0 ],
                                                                 corner-right => $_[ 1 ] ) };
    "{ $r1.area + $r2.area - $r1.overlapping-area( $r2 ) }".say;
}

```
