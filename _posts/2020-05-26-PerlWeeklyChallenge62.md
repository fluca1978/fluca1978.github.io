---
layout: post
title:  "Perl Weekly Challenge 62: queens and email addresses"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 62: queens and email addresses

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 62](https://perlweeklychallenge.org/blog/perl-weekly-challenge-062/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation here in Italy is slowly coming back to normal: several places have openened the last week and while we are still forced to stay at home as much as possible, we can move a little more freely.
<br/>
I have visited my mom twice in the last week, and she came to visit me too.
<br/>
I'm stil working from home, this is week number 12 I'm at home.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


### Olivia
Unluckily, our cat Olivia had a car accident, did undergo a surgery to fix with screws and a fixed external part on her hips.
<br/>
She is at home right now, but not able to move and needs help for feeding. The last check was good, and we could remove the collar the next week. We hope at mid June she can undergo the final surgery to remove the external part on her back.
<br/>
My and my wife are quite sad, because the process is going to be very long and we suffer in not having her jumping around the house.


<a name="task1"></a>
## PWC 62 - Task 1

The first task was very simple, so simple that I missed in the first run!
<br/>
You were asked to sort a list of emails by domain first, and by username later on.
<br/>
This can be solved chained two `sort` method invocations:

```perl6
@emails = @emails.sort( *.split( '@' )[ 0 ] ).sort( *.split( '@' )[ 1 ].lc );
```

<br/>
I got it wrong in the first run because you have to sort first for the last thing that needs to be sorted out, so in the first run I sorted first by domain and then by usernam, that lead to the wrong result.
<br/>
The idea is simple: apply `sort` splitting every email address by `@` and taking the first part (username), this will produce a list of emails that are going to be sorted again on the second part of th split against `@`. The `.lc` ensures the last sorting is case insensitive.
<br/>
<br/>


### Bonus
The challenge also asked for a bonus: printing only unique emails if a `-u` flag was specified.
<br/>
I changed the `MAIN` signature to include a `:$u` boolean optional flag, and this is the easy part. Then, if the flag is active, I `push` every email (lowecased) into a new array only if such array already does not contain such email in lowercase. The end result is an array of unique lower case emails.


```perl6
if $u {
    my @unique-emails;

    for @emails -> $email {
        @unique-emails.push: $email.lc if ( ! @unique-emails.grep( $email.lc ) );
    }

    say @unique-emails.join( "\n" );
}
```


<a name="task2"></a>
## PWC 62 - Task 2

The second task was really hard for me to solve: the placement of queens on a multidimensional cubical chessboard.
It was so hard to understand, that my implementation clearly reflect how I proceeded step by step to get something that could possibly be a solution.
<br/>
In the beginning, I decided to implement the chessboard as a cube hold by a list with `True` values on every square that can be occupied by a queen, and `False` on squares that cannot be used. A queen is placed into a square those value is turned into the string `QUEEN`.
<br/>
Therefore, the list is created as:

```perl6
sub MAIN( Int $dimension = 3 ){
    my @chessboard = [[True xx $dimension] xx $dimension] xx $dimension;
...
```

I [asked for some help on IRC](https://colabti.org/irclogger/irclogger_log/raku?date=2020-05-25#l303){:target="_blank"} because I was unable to get a mutable array using only `xx` and the comma list operator.
<br/>
After that, I implemented a brute force loop on every dimension:

```perl6
for 0 ..^ $dimension -> $height {
    for 0 ..^ $dimension -> $row {
        for 0 ..^ $dimension -> $column {

            # is the cell available?
            next if ! @chessboard[ $row ][ $column ][ $height ];

            # # place the queen
            place-queen( @chessboard, $row, $column, $height, $dimension );
        }
    }
}
```

If the cell is not set to `True` the queen cannot be placed, so I move to the next cell, otherwise I call `place-queen` to occupy the cell.
Occupying a cell with a queen means that all the directions where the queen can move must be set up to `False`, so that I will not consider them as available for another queen. The function is implemented in an ugly way:

```perl6
sub place-queen( @chessboard, $row, $column, $height, $dimension ){

    for 0 ..^ $dimension {
        @chessboard[ $row ][ $_ ][ $height ]    = False;
        @chessboard[ $row ][ $column ][ $_ ]    = False;
        @chessboard[ $_ ][ $column ][ $height ] = False;
    }

    # diagonal (only on one level)
    for 0 ..^ $dimension {
        @chessboard[ $row + $_ ][ $column + $_ ][ $height ] = False if ( $row + $_ < $dimension && $column + $_ < $dimension);
        @chessboard[ $row - $_ ][ $column - $_ ][ $height ] = False if ( $row - $_ >= 0 && $column - $_ >= 0 );
        @chessboard[ $row - $_ ][ $column + $_ ][ $height ] = False if ( $row - $_ >= 0 && $column + $_ < $dimension );
        @chessboard[ $row + $_ ][ $column - $_ ][ $height ] = False if ( $row + $_ < $dimension && $column - $_ >= 0 );
    }

    @chessboard[ $row ][ $column ][ $height ] = 'QUEEN';
}
```

In the beginning I invalidate the straight lines, then the diagonal (but only in one plain layer, because it is not clear to me if all diagonals in the cube have to be considered) and last I place the queen in the specified cell.
<br/>
<br/>
To show the progress and the final result, I've implemented a simple function that displays where the queens are and mark with an `x` other cells:

```perl6
sub show-chessboard( @chessboard, $dimension ) {
    for 0 ..^ $dimension -> $height {
        say "Layer $height";

        for 0 ..^ $dimension -> $row {
            for 0 ..^ $dimension -> $column {
                given @chessboard[ $row ][ $column ][ $height ] {
                    when Str { print "\t ", @chessboard[ $row ][ $column ][ $height ]; }
                    default  { print "\t  x  "; }
                }

            }

            print "\n";
        }
    }
}

```

<br/>
<br/>
The final result, for example with 3 cells per side is:

```perl6
% raku ch-2.p6
Layer 0
         QUEEN    x       x  
          x       x      QUEEN
          x       x       x  
Layer 1
          x      QUEEN    x  
          x       x       x  
         QUEEN    x       x  
Layer 2
          x       x      QUEEN
         QUEEN    x       x  
          x       x       x  

```

<br/>
I'm not proud of this solutio, since it exploits too much nested `for` loops, but so far it is the only one solution that comes to my mind.
