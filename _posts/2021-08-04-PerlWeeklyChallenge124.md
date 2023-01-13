---
layout: post
title:  "Perl Weekly Challenge 124: diffcult women!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 124: difficult women!

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 124](https://perlweeklychallenge.org/blog/perl-weekly-challenge-124/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 124 - Task 1

This task was about displaying the *venus sign* using ASCII-art. In the beginning I was interested in doing this, so I used an *here doc* string to print the venus symbol, but that was a very ugly solution.
<br/>
With a little more time, I decided to create a `Line` class that contains the information about how to print each line, that is it knows which symbol to use, where to start, where to end and how many chars to print. In particular, the `$!pad-with-blanks` determines if the space between `$!start` and `$!end` must be filled with blanks of with repetitions of `$!char`.
<br/>
Therefore, the program results in the construction of an array of `Line`s, and then an iteration of printing each entry thru the `draw` class method:

<br/>
<br/>
```raku
class Line {
    has $.char = '^';
    has $.start = 0;
    has $.end   = 0;
    has $.pad-with-blanks = True;

    method draw() {
        my $line = ' ' x $!start ~  $!char;
#        $line ~= $!start ~ "<>" ~ $!end;
        if ( $!pad-with-blanks ) {
            $line ~= ' ' x ( $!end - $!start - 1 ) ~ $!char;
        }
        else {
            $line ~= $!char x ( $!end - $!start );
        }

        say $line;

    }
}


sub MAIN() {

    my @lines;
    my $size = 5;
    my ( $start, $end ) = $size, $size * 2 - 1;
    @lines.push: Line.new( start => $start, end => $end, pad-with-blanks => False );
    while ( $start > 0 ) {
        @lines.push: Line.new( start => --$start, end => ++$end );
    }

    # add three lines
    @lines.push: Line.new( start => 0, end => $end ) for 1 .. 3;

    # decreasing part
    while ( $start < $size ) {
        @lines.push: Line.new( start => ++$start, end => --$end );
    }

    # final line
    @lines.push: Line.new( start => $start, end => $end, pad-with-blanks => False );

    # add three lines with a single char
    @lines.push: Line.new( start => $start + $size / 2, end => $start + $size / 2, pad-with-blanks => False ) for 1 .. 3;
    @lines.push: Line.new( start => $start, end => $end, pad-with-blanks => False );
    @lines.push: Line.new( start => $start + $size / 2, end => $start + $size / 2, pad-with-blanks => False ) for 1 .. 3;

    .draw for @lines

}

```
<br/>
<br/>


<a name="task2"></a>
## PWC 124 - Task 2

The second task was a little harder, because it involved some math.
The idea was to divide an array into two sub-arrays (called *sets*) so that the difference between the sum of the arrays was as least as possible.

<br/>
<br/>
```raku
sub MAIN( *@S where { @S.elems == @S.grep( * ~~ Int ).elems && @S.elems > 0 } ) {
    my $split-pos = do given @S.elems %% 2 {
        when .so { @S.elems / 2 }
        default { ( @S.elems - 1 ) / 2 }
    };

    my $min-diff;
    my $sum = ( @S.sum / 2 ).Int.floor;
    my @solution;

    for @S.combinations: $split-pos -> @current-set {
        my $diff = ( $sum - @current-set.sum ).Int.abs;
        if ( ! $min-diff || $min-diff > $diff ) {
            @solution = @current-set.clone;
            $min-diff = $diff;
        }
    }

    # assume each number appears one and only one
    my @anti-solution;
    for @S {
        @anti-solution.push: $_ if ! @solution.grep( $_ );
    }

    say "Sets are { @solution.join( ',' ) } and { @anti-solution.join( ',' ) }";
}


```
<br/>
<br/>

 Initially I do a check on the input `@S` array to ensure is made by integers and only integers. Then I decide how many parts of the sets must be fit computing the size of `@S` and seeing if it is odd or even.
 <br/>
 Initially I started using a `permutations` against the whole array, but it took too much time even if executed in parallel, so I decided to compute an initial sum of the `@S` array, and compute its half: the subsets should tend to such value in order to reduce the difference.
 <br/>
 Then I computed the `combinations` of the `@S` considering only an half of it (or half plus 1), and for each sub-array produced I computed the `sum` and the difference with the overall sum to which a subset should tend. When the difference has its lower value, the subset is composed into `@solution`. In order to get the remaining part of the `@S`, that is the other set, I excluded every number within the `@S` and `@solutions` and built the second set into `@anti-solution` (ok, the name is ugly).
 <br/>
 This approach has two main drawbacks:
 - it works if and only if a number appears only once in the initial `@S`, or the final `grep` to build `@anti-solution` has to be changed;
 - it does not find all the available solutions, just the former one in the case there are more than one.
