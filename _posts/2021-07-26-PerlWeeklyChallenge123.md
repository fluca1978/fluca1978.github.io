---
layout: post
title:  "Perl Weekly Challenge 123: squares and ungly numbers!" 
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 123: squares and ugly numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 110](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0110/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 123 - Task 1

The first task was about computing the n-th *ugly number*, where an ugly number is a number prime by means of `2`, `3` and `5`. This was quite simple, and since I didn't know in advanced what n-th number was asked, I decided to implement as a `lazy gather` form.
<br/>
The application is as follows:

<br/>
<br/>
```raku

sub is-ugly( Int $n ) {
    # short circuit!
    return True if $n == any( 2, 3, 5 );

    return $n %% 2 || $n %% 3 || $n %% 5;
    
}

sub MAIN( Int $n where { $n > 1 } ) {
    my  @ugly-numbers = lazy gather {
        for 1 .. Inf {
            take $_ if $_ == 1;
            take $_ if is-ugly( $_ );
        }  
    } 

    
    @ugly-numbers[ $n - 1 ].say;
}

 ```
<br/>
<br/>

The `is-ugly` function accepts a number and checks if it is ugly, that is if it can be divided by `2`, `3` and `5`.
<br/>
The `MAIN` defines an `@ugly-numbers` array that is made by a `lazy` loop from `1` to the infinite. I don't understand why, but `1` is ugly, so I manually inserted it as a special `take` branch. On the other cases, the `is-ugly` function is the way to test if the current number is ugly or not, and if it is the number is `take`n.
<br/>
At the end, the computed `$n`-th value is printed out.


<a name="task2"></a>
## PWC 123 - Task 2

The second task was about understanding if, given four points (in a two dimensional space), they form a square.
This is a math problem, that I decided to implement with a `Point` class, that contain a `distance` method that computes the distance from the current point and another one.

<br/>
<br/>
```raku
class Point {
    has Int $.x;
    has Int $.y;

    method distance( Point $other ) {
        return ( ( $other.x - $!x ) * ( $other.x - $!x )
        + ( $other.y - $!y ) * ( $other.y - $!y ) ).sqrt;
    }
}

```
<br/>
<br/>
The `MAIN` accepts all the coordinations of the points, and builds up the `Point` objects.
<br/>
With the `Point`s in place, I can check if the distance between the points are the same (i.e., the square has every segment of the same length). Then I can check if the diagonals have the same length, last if the segments make a square angle. To perform the last test, I assume that the square can be split into two square triangles, and if that is true the points make a square.
<br/>
<br/>
```raku
sub MAIN( Int $x1, Int $y1,
          Int $x2, Int $y2,
          Int $x3, Int $y3,
          Int $x4, Int $y4 ) {


    my Point $p1 = Point.new( :x( $x1 ), :y( $y1 ) );
    my Point $p2 = Point.new( :x( $x2 ), :y( $y2 ) );
    my Point $p3 = Point.new( :x( $x3 ), :y( $y3 ) );
    my Point $p4 = Point.new( :x( $x4 ), :y( $y4 ) );

    # check if the distances are the same
    my $ok-length = $p1.distance( $p2 ) == $p2.distance( $p3 ) == $p3.distance( $p4 ) == $p4.distance( $p1 );

    '0'.say and exit if ! $ok-length;

    # check if the diagonals have the same distance
    my $ok-diags = $p1.distance( $p3 ) == $p2.distance( $p4 );

    '0'.say and exit if ! $ok-diags;

    # check if the P1, P2, P3 and P1, P4, P3 are on square triangle
    my $l = $p1.distance( $p2 );
    my $d = ( 2 * $l * $l ).sqrt;
    my $ok-triangle = $d == $p1.distance( $p3 ) && $d == $p4.distance( $p2 );

    '0'.say and exit if ! $ok-triangle;
    '1'.say;
}

```
<br/>
<br/>
After every single test, I quit the program immediatly if the test fails.
<br/>
Please note that points `p1` and `p3` are opposite, as well as `p2` and `p4`.
