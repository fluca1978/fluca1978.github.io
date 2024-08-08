---
layout: post
title:  "Perl Weekly Challenge 281: Let's Play Chess!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 281: Let's Play Chess!

This post presents my solutions to the [Perl Weekly Challenge 281](https://perlweeklychallenge.org/blog/perl-weekly-challenge-281/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 281 - Task 1 - Raku](#task1)
- [PWC 281 - Task 2 - Raku](#task2)



# Raku Implementations

<a name="task1"></a>
## PWC 281 - Task 1 - Raku Implementation

The first task was about finding out if a given position on a chessboard was light or not.



<br/>
<br/>
```raku
sub MAIN( Str $coordinates where { $coordinates ~~ / ^ <[a .. h]> <[ 1 .. 8 ]> $/ } ) {
    my %chessboard;
    my $value = False;
    for 1 .. 8 -> $row {
		$value = $row %% 2 ?? True !! False;
		for 'a' .. 'h' -> $column {
			%chessboard{ $column ~ $row } = $value;
			$value = ! $value;
		}
    }

    %chessboard{ $coordinates }.say;

}

```
<br/>
<br/>

I lazily created a nested loop over `%chessboard`, which is initialized with `True` for light positions and `False` for black ones.
Therefore, it becomes really simple to see if a given position is black or white.


<a name="task2"></a>
## PWC 281 - Task 2 - Raku Implementation

This has been a very complex task: find out the shortest path for a knight to move from a position to another.


<br/>
<br/>
```raku
sub compute-end-points( $start ) {
    my @end-points;
    my @valid-columns = 'a' .. 'h';

    $start ~~ / ^ $<column>=( <[a..h]> ) $<row>=( <[1..8]> ) $ /;
    my $row = $<row>.Int;
    my $column = $<column>.Str;

    return Nil if ( $row > 8 || $row < 1 || ! @valid-columns.grep( { $_ ~~ $column } ) );

    # at which index is the current column labeled with a letter?
    $column    = @valid-columns.grep( { $_ ~~ $column }, :k )[ 0 ];

    my @moves = [ +1, -2 ], [ +1, +2 ],
		[ +2, -1 ], [ +2, +1 ],
		[ -1, -2 ], [ -1, +2 ],
		[ -2, -1 ], [ -2, +1 ];

    @end-points = @moves.map(
	{  [ @valid-columns[ $_[ 0 ] + $column ], $_[ 1 ] + $row ] } )
		   .grep( { $_[ 0 ] && @valid-columns.grep( * ~~ $_[ 0 ] ) && 1 <= $_[ 1 ] <= 8 } )
		   .map( { $_[ 0 ] ~ $_[ 1 ] } );
		   ;
    return @end-points;
}


sub MAIN( Str $start,
	  Str $end,
	  :$verbose
			  where { $start !~~ $end
				  and $start ~~ / ^ <[a..h]> <[1..8]> $ /
				  and $end ~~ / ^ <[a..h]> <[1..8]> $ / }
			   ) {


    my @end-points = compute-end-points( $start );

    my @paths;
    @paths = @end-points.map( $start ~ '-' ~ * );

    my @new-paths;
    my $found = False;
    my $debug = 0;
    while ( ! $found ) {
		for @paths.sort -> $current-path  {
		    my $next = $current-path.split( '-' )[ * - 1 ];
		    my @ends = compute-end-points( $next ).grep( { $_ ne $start && $_ ne $next } );
		    @new-paths.push: $current-path ~ '-'  ~ $_ for @ends;
		}

		@paths = @new-paths;
		@new-paths = ();
		$found = @paths.grep( * ~~ / $end $ / );

    }



    my %possible-moves;
    for @paths.grep( * ~~ / $end $ / ) {
		%possible-moves{ $_.split( '-' ).elems }.push: $_;
    }

    my $min-moves = %possible-moves.keys.min;
    $min-moves.say;
    if ( $verbose ) {
		"possible moves:".say;
		%possible-moves{ $min-moves  }.sort.join( "\n" ).say;
    }
}


```
<br/>
<br/>

I define a function `compute-end-points` that, given a starting position, computes where the kight can be that, in a case where all the endings are still on the chessboard, means 8 possible positions.

Then, in the `MAIN`, I start from the beginning position and compute the initial `@end-points` list. Then I do iterate until I find the final destination, appending to a list of positions, the ending points. So the list `@new-paths` is built as `start-end1, start-end2, ...` and then, in the next iteration, is expanded to `start-end1-subend1, start-end1-subend2, ... , start-end8-subend1, ...`.

Once I've the list of all possible places the knight can reach, I `grep` thos that end with the `$end` wanted position, splitting into places and computing the minimum value of moves. This can lead to more than one sequence, so I compare them into an hash keyed at the number of moves.
