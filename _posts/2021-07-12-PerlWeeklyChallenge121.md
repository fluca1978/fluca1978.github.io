---
layout: post
title:  "Perl Weekly Challenge 121: bits and salesman"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 121: bits and salesman

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 121](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0121/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 121 - Task 1

The first task was (again) about binary representation of numbers: given an integer number, you have to flip its `n-th` bit and print out the decimal representation. Since I don't remember how to flip a bit, I solved it with the ternary operator:

<br/>
<br/>
```raku
sub MAIN( Int $m where { 0 <= $m <= 255 }
          , Int $n where { 1 <= $n <= 8 } ) {
    my @bits = '%08d'.sprintf( $m.base( 2 ) ).split( '', :skip-empty );

    @bits[ * - $n ] = @bits[ * - $n ] == 1 ?? 0 !! 1;

    @bits.join.parse-base( 2 ).say;
}
```
<br/>
<br/>



The `@bits` array contains 8 digits in binary format, and then I flip the `$n` to last one, printing out the decimal value by means of the `parse-base` function.


<a name="task2"></a>
## PWC 121 - Task 2

I hate the salesman problem! I always hated it, even when I was studying it at the university. And the second task was about solving the problem of the salesman given a matrix (a square matrix) with the distances between the cities.
<br/>
First of all, I implemented a function that, given a row of distances from the current city, finds out the minimum city to go to:

<br/>
<br/>
```raku
sub find-city-path( @matrix, $row-number, $allow-zero ) {
    my @row = @matrix[ $row-number ].List;
    my $min-distance = @row.max + 1;
    my $current-city = 0;
    my $city = -1;


    for @row -> $distance {

        $city++;
        next if $distance == 0 && ! $allow-zero;

        if ( $distance < $min-distance ) {
            $current-city = $city;
            $min-distance = $distance;
        }

    }



    return [ $min-distance, $current-city ];
}

```
<br/>
<br/>

Given the overall `@matrix` of distances, as well as `$row-number`, that is the current city we are in, the function computes the next city to visit and the distance and returns it as an array. The `$allow-zer` is a special flag to allow the `0` distance to be the minimum, thus selecting the current city as a starting point. This is used to select the very fist city.
<br/>
<br/>
Then, there is the need to iterate over all the cities, and in particular to iterate on the length of the matrix, that being squared, is the length of its first row. On every iteration, the next city is selected and the `@path` array stores the distances of the cities:

<br>
<br>
```raku
sub MAIN() {
    my @matrix = [0, 5, 2, 7]
        , [5, 0, 5, 3]
        , [3, 1, 0, 6]
        , [4, 5, 4, 0];

    my @path;
    my $cities-left = @matrix[ 0 ].elems;


    my ( $distance, $city ) = Nil, 0;
    while ( $cities-left > 0 ) {
        ( $distance, $city ) = find-city-path( @matrix, $city, $cities-left == @matrix[ 0 ].elems );
        @path.push: $distance;
        $cities-left--;


    }

    @path.push: 0;

    @path.say;
}

```
<br/>
<br/>

It is too much code for being Raku, but I don't want to spend more brainpower on this because really I hate this kind of problem!
<br/>
Please note that, on the end, there is a distance of zero because we are supposed to be on the last city to visit (that is not the same as the last city in the array of cities!).
