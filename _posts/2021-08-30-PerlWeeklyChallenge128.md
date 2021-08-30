---
layout: post
title:  "Perl Weekly Challenge 128: too much complex?" 
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 128: to much complex?

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
## PWC 128 - Task 1

The first task was very complex to solve, at least according to me. I'm not proud of this solution, because it uses extensively nested loops, that is somehow not the way Raku should work!
<br/>
The task was requiring you to find out the largest sub-matrix of a given matrix made by `0` and `1`, so that the sub.matrix is made by only `0` elements.
<br/>
I decided to go like this:
- first I map every row of the matix so that each value corresponds to the counting of zeroes from this position to the rightmost one;
- I do loop over this new matrix and, for every non-zero value (that means there is a zero from here), I look at the next rows to see if there's also a zero in the same column. I end up searching for as quick as the next row does not contain a zero;
- the minimum between all the counters in subsequent rows gives me the width of the sub-matrix;
- I place every area of the matix, along with starting and ending position, into an array so that I can get back information later. In fact, *there could be more than one matrix that has the same largest area*;
- I then print every value found in the information array.

<br/>
The program results as follows:


<br/>
<br/>
```raku
sub MAIN() {
    my @matrix = [ 1 ,0, 0, 0, 1, 0 ],
                 [ 1 ,1, 0, 0, 0, 1 ],
                 [ 1, 0, 0, 0, 0, 0 ];

    my @zeroes;

    for 0 ..^ @matrix.elems -> $current-row {
        for 0 ..^ @matrix[ $current-row ].elems  -> $current-column {
            my $zeroes-count    = 0;
            my $column          = $current-column;
            my $previous-column = $current-column;
            while ( $column < @matrix[ $current-row ].elems && $column - $previous-column <= 1 ) {
                $zeroes-count++ if @matrix[ $current-row ][ $column ] == 0;
                last if @matrix[ $current-row ][ $column ] != 0;
                $previous-column = $column++;
            }

            @zeroes[ $current-row ][ $current-column ] = $zeroes-count;
        }
    }

    say @zeroes;

    my $rows = 0;
    my $cols = 0;
    my @sub-matrix;
    my $max  = 0;
    for 0 ..^ @zeroes.elems -> $current-row {
        for 0 ..^ @zeroes[ $current-row ].elems -> $current-column {
            next if @zeroes[ $current-row ][ $current-column ] == 0;
            $cols = @zeroes[ $current-row ][ $current-column ];
            $rows = 1;
            
            
            for $current-row + 1 ..^ @zeroes.elems -> $next-row {
                $rows = 0 and last if @zeroes[ $next-row ][ $current-column ] == 0;
                $rows++ if @zeroes[ $next-row ][ $current-column ] != 0;
                $cols = min( $cols, @zeroes[ $next-row ][ $current-column ] );
            }

            
            $max = $rows * $cols and @sub-matrix = () if $rows * $cols > $max;
            @sub-matrix.push: [ $rows * $cols, $current-row, $current-column, $current-row + $rows - 1, $current-column + $cols - 1 ] if $rows * $cols > 0 && $rows * $cols >= $max;

            


        }
    }

    "{ $_[ 0 ] } zeroes starting from <{ $_[ 1 ] }, { $_[ 2 ]}> to <{ $_[ 3 ] }, { $_[ 4 ]}>".say for @sub-matrix;
    

    

}
```
<br/>
<br/>

In the first loop, I build up a `@zeroes` matrix where each line has values that corresponds to the counting of subsequent zeroes moving to the right.
<br/>
In the second nested loop, I do move on each line of the `@zeroes` array searching for a non-zero element. Once I found it, I open a new submatrix (`$rows = 1`), and then go down within the same column to count how many subsequent rows there are with non-zero counting. During this process, I compute the `$cols` value as the minimum between all the counters I found in the rows, so that I know how large the submatrix is going to be.
In the end, when no more rows are there, I compute the area of the submatrix as `$rows * $cols` and store them as the new `$max` value, and then I push to the `@sub-matrix` array.
<br/>
Please note that every time a new *larger* sub matrix is found, the `@sub-matrix` array is reset, so that in the end it will contain only the wider matrix information.


<a name="task2"></a>
## PWC 128 - Task 2

The second task was a little simpler: given a train timetable, find out how many *parallel* platform are needing to avoid collisions.
<br/>
The time table is kept in a set of two array, where the same index refers to the same train.
<br/>
I decided to build a `Train` class, with an ad-hoc constructor that given the timetable strings, defines the train by hours and minutes:
<br/>
<br/>
```raku

class Train {
    has Int $.hour-arrival;
    has Int $.hour-departure;
    has Int $.minute-arrival;
    has Int $.minute-departure;


    submethod BUILD( Str :$arrival, Str :$departure ) {
        $arrival ~~ / (\d+) ':' (\d+) /;
        ( $!hour-arrival, $!minute-arrival ) = $/[ 0 ].Int, $/[ 1 ].Int;
        $departure ~~ / (\d+) ':' (\d+) / ;
        ( $!hour-departure, $!minute-departure ) = $/[ 0 ].Int, $/[ 1 ].Int;
    }

    method collide( Train $other-train ) {
        my $this-arrival   = $!hour-arrival * 60 + $!minute-arrival;
        my $this-departure = $!hour-departure * 60 + $!minute-departure;
        my $other-train-arrival   = $other-train.hour-arrival * 60   + $other-train.minute-arrival;
        my $other-train-departure = $other-train.hour-departure * 60 + $other-train.minute-departure;

        $this-arrival <= $other-train-arrival && $other-train-departure <= $this-departure;
    }
}

```
<br/>
<br/>

The `BUILD` constructor does some simple regular expression matching and builds the hour and minutes as integers.
The `collide` method does the magic: computes the minutes for arrival and departure of the instance object and of the other train to compare to, and then returns `True` if there is a collision. Of course, this assumes the time table is expressed in 24-hours representation.

<br/>
<br/>
Then, the `MAIN` builds an array of `Train` instances and scans with a loop thru the array to see how many collisions there are. Since at least one platform is required, even if no collision happen, the total number of platforms is given by `1` plus the number of collisions:


<br/<
<br/>
```raku
sub MAIN() {
    my @arrivals = '10:20', '11:00', '11:10', '12:20', '16:20', '19:00';
    my @departures = '10:30', '13:20', '12:40', '12:50', '20:20', '21:20';


    my @trains.push: Train.new: arrival => @arrivals[ $_ ], departure => @departures[ $_ ] for 0 ..^ @arrivals.elems;

    my $collisions = 0;
    for 0 ..^ @trains.elems -> $current-train {
        $collisions++ if @trains[ $current-train ].collide( @trains[ $_ ] ) for $current-train + 1 ..^ @trains.elems;
    }
    
    "Required platforms: { $collisions + 1 }".say;


}

```


<br/>
<br/>
