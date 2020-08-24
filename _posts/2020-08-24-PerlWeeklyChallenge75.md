---
layout: post
title:  "Perl Weekly Challenge 75: nested loops"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 75: nested loops

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 75](https://perlweeklychallenge.org/blog/perl-weekly-challenge-075/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 75 - Task 1

You have an infinite set of finite coins, and you need to find out all the available sets that can provide a specific sum.
My first attempt at this task was to *permutate* the list of available coins, but that resulted in a *long* computation.
Therefore, I decided to implement it via a set of nested loops.
<br/>
First of all, let's see the `MAIN` entry point:

<br/>
<br/>
```raku
sub MAIN( Int $S where { $S > 0 },
          *@C where { @C.grep( * ~~ Int ).elems == @C.elems } ) {
    say "Coins are { @C } that must give sum $S";

    my @solutions;

    @solutions = find-solutions( coins => @C, sum => $S );

    @solutions.join( "\n" ).say;

}
```
<br/>
<br/>


All the boring stuff is done by the `find-solutions` method, that is described below.


<br/>
<br/>
```raku

sub find-solutions( Int :$sum, :@coins ) {
    my @solutions;

    # add all the 'one coin' solutions
    @solutions.push: [ $_ xx ( $sum / $_ ) ] for @coins.grep( $sum %% * );


    # now inspect all the other cases
    for @coins.grep(  $sum !%% * ).sort( { $^b <=> $^a } ) -> $current-coin {
        next if $current-coin > $sum;
        my @current-solution;
        @current-solution.push: $current-coin;

        for @coins.grep( * !~~ $current-coin ).sort( { $^b <=> $^a } ) {
            my $current-sum = [+] @current-solution;
            # try to add the same number over and over again
            while ( ( $current-sum + $_ ) <= $sum ) {
                @current-solution.push: $_ ;
                $current-sum = [+] @current-solution;
            }

            if ( $current-sum == $sum  ) {
                # use a new array to clone it, so I can delete the last value and continue over
                @solutions.push: Array.new: @current-solution.sort;
                @current-solution[ *-1 ]:delete;
                next;
            }
        }

    }


    # last step: decompose every number in a solution into other numbers
    my @decomposed-solutions;
    for @solutions -> @current-solution {
        for 0 ..^ @current-solution.elems -> $switching-index {

            my $current-coin = @current-solution[ $switching-index ];
            next if $current-coin ~~ 1;

            for @coins.grep( $current-coin %% * ) {
                next if $_ ~~ $current-coin;
                my @new-solution = Array.new: @current-solution;
                @new-solution[ $switching-index ]:delete;
                @new-solution[ $switching-index ] = | ( $_ xx ( $current-coin / $_ ) );
                @decomposed-solutions.push: [ @new-solution.sort ];
            }
        }
    }


    # now build something unique
    my %unique-solutions;
    for @decomposed-solutions {
        %unique-solutions{ "{ $_ }" } = $_;
    }

    for @solutions {
        %unique-solutions{ "{ $_ }" } = $_;
    }


    %unique-solutions.values;
}

```
<br/>
<br/>

The first step is to add all the possibile *easy solutions*: all the repetitions of coins that provide the exact sum required, and this is done using the list repetition `xx` and the remainder `%%` one.
<br/>
Then I search for all the coins that have not been used so far, and sorting from the greatest to the smallest, I try to compose a set of sums with the all remaining coins. The sum is performed as follows:
- add the same coin over and over until the sum is done;
- add another smallest coin when it is not possible to sum the same coin because there is an overflow.
This makes the `@solutions` array made by pretty much *unique* sequences of sums.
<br/>
Last step is decomposition: every solution can be rebuilt as a sub-sum of smallest coins, therefore I iterate on every solution and try to substitute every number with its *smaller coin sum* equivalent.
<br/>
In the end, I use an hash to get out duplicated solutions, and return all the values to the caller.


<a name="task2"></a>
## PWC 75 - Task 2

The second task was about finding out the biggest rectangle that is laying in an histogram.
First of all, I declared a couple of classes for representing a rectangle and an histogram:


<br/>
<br/>
```raku
class Histogram {
    has Int $.column;
    has Int $.height;

    method Str() { "Histogram $!height Column $!column"; }
}


class Rectangle {
    has Int $.height;
    has Int $.base;

    method area() { $!height * $!base }
    method Str()  { "Rectangle with base $!base and height $!height ($!base x $!height)" }
}

```
<br/>
<br/>


The first step the `MAIN` has to do is to build the array of historgrams. Then I need to loop over the histograms and try to get the biggest rectangle for every height. The trick part here is that if the next historgram has a lower height than the rectangle is done, otherwise I continue to increment it.


<br/>
<br/>
```raku

sub MAIN( *@A
          where { @A.grep( * ~~ Int ).elems == @A.elems  && @A.grep( *.Int >= 1 ) } ) {
    say "Numbers are { @A }";

    my Histogram @histograms;
    my $column = 1;
    for @A {
        @histograms.push: Histogram.new: column => $column, height => $_.Int;
    }


    my Rectangle @rectangles;
    my ( $current-height, $current-width ) = Nil, Nil;

    for 0 ..^ @histograms.elems -> $current-index {
        ( $current-height, $current-width ) = @histograms[ $current-index ].height, 0;


        # go backwards to the first histogram that has a good height
        my $starting-index = $current-index;
        while ( $starting-index > 0 &&  @histograms[ $starting-index ].height >= $current-height ) {
            $starting-index--;
        }

        for $starting-index ..^ @histograms.elems {
            next if $_ < $current-index && @histograms[ $_ ].height < $current-height;
            if @histograms[ $_ ].height < $current-height {
                last;
            }
            else {
                $current-width++;
            }
        }

        @rectangles.push: Rectangle.new( height => $current-height,
                                         base => $current-width );
    }



    # get the first one with the biggest area
    @rectangles.sort( { $^b.area <=> $^a.area } ).first.put;


}

```
<br/>
<br/>

Last, I sort the rectangles depending on their `area` and pick the `first` one (reversed order), so that I can print the biggest one.


### Bonus track

The bonus track was to graph the histograms. I did a little more and also emphasized the biggest rectangle.
To achieve this, I added a `Range` named `$!columns` in the `Rectangle`, so that I can keep track of which columns the rectangle occupies (they need to be contiguos).
<br/>
Then I designed a `graph` function that accepts an optional `Rectangle` and displays the stringified version of every histogram, from the highest to the shortest one, selecting a simble to display the rectangle if present.

<br/>
<br/>
```raku
sub graph( Histogram :@histograms, Rectangle :$rectangle? ) {
    my @lines;
    my $max-height = max @histograms.map( { .height } );


    while ( $max-height > 0 ) {
        my @line;
        @line.push: $max-height ~ '| ';

        for @histograms {
            my $column = .column;
            my $height = .height;
            my $to-print = $rectangle
            && $column ~~ $rectangle.columns
            && $rectangle.height >= $max-height
            ?? ' X' !! ' #';
            @line.push: .height >= $max-height ?? $to-print !! '  ';
        }

        @lines.push: @line.join;
        $max-height--;
    }

    @lines.push: '---' x @histograms.elems;
    @lines.push: '    ' ~ @histograms.map( { .column } ).join( ' ' );
    @lines;
}

```
<br/>
<br/>


and the construction loop for the `Rectangle` has changed to the following one:

<br/>
<br/>
```raku
for $starting-index ..^ @histograms.elems {
    next if $_ < $current-index && @histograms[ $_ ].height < $current-height;
    if @histograms[ $_ ].height < $current-height {
        last;
    }
    else {
        $current-width++;
        $start-column = min $_ + 1 , $start-column;
        $end-column   = max $_ + 1, $end-column;
    }
}

@rectangles.push: Rectangle.new( height => $current-height,
                                 base => $current-width,
                                 columns => $start-column .. $end-column  );
```
<br/>
<br/>
