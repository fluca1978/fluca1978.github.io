---
layout: post
title:  "Perl Weekly Challenge 126: counting mines!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 126: counting mines!

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 126](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0126/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 126 - Task 1

The first task was very simple, or at least I was quick at solving it: count all the numbers in the sequence from `1` to a given `$N` that do not include any `1` digit.
<br/>
It is easy with `grep`:

<br/>
<br/>
```raku
sub MAIN( Int $N ) {
    ( 1 .. $N ).grep( * !~~ / 1 / ).elems.say;
}
```
<br/>
<br/>

The idea is to produce the sequence of numbers, then `grep` for searching for a regular expression that does not match any `1`, then I count how many elements are in the array by means of `elemns` and print them out.

<a name="task2"></a>
## PWC 126 - Task 2

The second task was about rewriting the mine swiper game. The task was not hard, but I solved with a few nested loops.
<br/>
First of all, let's do a couple of utility functions:
- `is-mine` accepts the array of input and the position (as row and column), and returns true if the position is a mine (i.e., an `x`);
- `compute-mines` accepts the same input and tries to move on near positions to see how many mines there are.



<br/>
<br/>
```raku
sub is-mine( @input, $row, $column ) {
    return @input[ $row ][ $column ] ~~ 'x';
}


sub compute-mines( @input, $row, $column ) {
    my $counter = 0;

    # see where we can move
    my ( $can-left, $can-right, $can-up, $can-down ) =
         $column > 0, $column < @input[ 0 ].elems, $row > 0, $row < @input.elems;


    # left, if possible
    $counter++  if ( $can-left && is-mine( @input, $row, $column - 1 ) );
    # up if possible
    $counter++  if ( $can-up && is-mine( @input, $row - 1, $column ) );
    # down if possible
    $counter++  if ( $can-down && is-mine( @input, $row + 1 , $column ) );
    # right if possible
    $counter++  if ( $can-right && is-mine( @input, $row, $column + 1 ) );


    # left up
    $counter++  if ( $can-left && $can-up && is-mine( @input, $row - 1, $column - 1 ) );
    # right up
    $counter++  if ( $can-up &&  $can-right && is-mine( @input, $row - 1, $column + 1 ) );
    # left down
    $counter++  if ( $can-left && $can-down && is-mine( @input, $row + 1, $column - 1 ) );
    # right down
    $counter++  if ( $can-down && $can-right && is-mine( @input, $row + 1, $column + 1 ) );

    # left up if possible


    return $counter;
}

```
<br/>
<br/>

The most complex part is within the `compute-mines`: in short the function checks if it can move up, down, left and right and diagonal directions without going over the array boundaries. If the move can be done, and in that position there is a mine, the `$counter` variable is incrementd.

<br/>
It is now turn to use the above functions in the main program:

<br/>
<br/>
```raku
sub MAIN() {
    my @input =
     qw/ x * * * x * x x x x /,
     qw/ * * * * * * * * * x /,
     qw/ * * * * x * x * x * /,
     qw/ * * * x x * * * * * /,
     qw/ x * * * x * * * * x /;

    my $rows = @input.elems;
    my $columns = @input[ 0 ].elems;




    my ( $current-row, $current-column ) = 0, 0;
    for 0 ..^ $rows -> $current-row {
        for 0 ..^ $columns -> $current-column {
            print is-mine( @input, $current-row, $current-column ) ?? 'x'
                      !! compute-mines( @input, $current-row, $current-column );
        }

        print "\n";
    }
}

```
<br/>
<br/>

As you can see, the `@input` array contains the mine field, and the first step is to compute the number of rows and columns, than to do a nested loop to print either the mine or the number computed by `compute-mines`.
<br/>
The final result is something like the following:

<br/>
<br/>
```shell
% raku ch-2.p6
x101x2xxxx
126224355x
0013x3x2x2
111xx41222
x113x2001x

```
