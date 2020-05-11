---
layout: post
title:  "Perl Weekly Challenge 60: Column Names and Compound Numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 60: bits and arrays

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 60](https://perlweeklychallenge.org/blog/perl-weekly-challenge-060/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation in Italy is ugly and to some extent desperate: it has been 10+ weeks since I'm closed into my house with my son and my wife, we cannot move and cannot go outside. Probably the soldiers will become to monitor the streets very soon this week.
<br/>
<br/>
I've visited my mum twice, but going there is not as simple as it could seem.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


### Olivia
Unluckily, our cat Olivia had a car accident last Friday and is now in hospital for a surgery.
<br/>
Me and my wife are really sad, and I lost at least two nights at sleeping. Luckily, our son is not accusing the problem and is convinced the cat will be quickly fine and at home.
<br/>
We hope, too. But my mind is totally away right now.


<a name="task1"></a>
## PWC 60 - Task 1

The first task was an *Excel column name and index translator*: given a column number the script must return the name as made by letters, while given a column name it must return the index number.
<br/>
I implemented it using a *two branch* application: one branch for the numeric input, and one for the letter input.
<br/>
With regard to the number branch:

```perl6
sub MAIN( $what  ) {
    my @letters = 'A' .. 'Z';
    my @column-name;


    # numeric value to be converted into a letter
    if $what ~~ Int && $what > 0 && $what <= 16_384 {
        my $column = $what.Int;


        while ( $column > @letters.elems ) {
            @column-name.push: @letters[ $column / @letters.elems  - 1 ];
            $column = $column % @letters.elems;
        }

        @column-name.push: @letters[ $column - 1 ];
        say "Cell $what is { @column-name.join }";
    }

```

The idea is that if the number of the column is greater than the number of available letters, 26, a new letter has to be pushed to the array of the letters `@column-name`. The last pass is to take the reminder (subtracted by one because of `A` being the `0` index).
<br/>
With regard to the letter branch:

```perl6
    elsif $what ~~ Str {
        my $column;
        my $multiplier = 0;
        # string, try to find the cell number
        for $what.comb.reverse -> $current_letter {
            $column += @letters.first( $current_letter, :k ) + 1 + ( @letters.elems - 1 ) * $multiplier++;
        }

        say "Cell $what is $column";
    }
```


The approach is the oppoite as the one shown before: I got the `.first` index of the letter, add one and multiply for the number of letters depeding on the loop counter.


<a name="task2"></a>
## PWC 60 - Task 2

The second task was related to extracing a sequence of numbers made by digits in a given array and that were `$x` digits in length and less than `$y` value.
<br/>
I implemented it as follows:

```perl6
    for ( 1 x $x ) - 1 ..^ $y {

        # see if the numbers "grep" the list
        my $found = 0;
        $found++ if $_ == any( @L ) for $_.comb;
        @LL.push: $_ if $_.comb.elems == $found;
    }

```

I start a sequence of all the numbers less than `$y` and that start from the `$x` length. Within the loop I count the number of digits found in `@L` using a *junction*. If the number of found elements is equal to the number of digits that made the number, it means that the number is made by only numbers in the `@L` array of available digits. In this case, the number is pushed to an array of collected good numbers `@LL`, that is then printed in the `MAIN` method ending.
