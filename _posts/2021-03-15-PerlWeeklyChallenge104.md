---
layout: post
title:  "Perl Weekly Challenge 104: recursion and picking"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 104: recursion and picking

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 104](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0104/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)





<a name="task1"></a>
## PWC 104 - Task 1
The first task was quite simple: it required to compute a *FUSC* sequence, that is something that can be computed recusrively. Since Raku allows for parameter value overloading of a function, why not use that?

<br/>
<br/>
```raku
multi sub fusc( 0 ) { 0 }
multi sub fusc( 1 ) { 1 }
multi sub fusc( $n where { $n > 1 } ) {
    return samewith( ( $n / 2 ).Int ) if $n %% 2;
    return samewith( ( ( $n - 1 ) / 2 ).Int ) + samewith( ( ( $n + 1 ) / 2 ).Int );
}


sub MAIN(){
    "fusc( $_ ) = { fusc( $_ )}".say for 0 .. 10;
}
```

The idea is to define a `fusc` function that will have fixed values for `0` and `1` and will recursively call itself by mean of `samewith` with different arguments. There is the need to add a cast to `Int` or the recursive call will not be able to find out the correct candidate.



<a name="task2"></a>
## PWC 104 - Task 2

The second task was about a game where the players have to pick 1 or 2 or 3 tokens at the same time, and the one that picks the last token from a group of 12 wins. I've implemented it using an array of `@tokens` from which I remove a random number of tokens at every round, for two players:

<br/>
<br/>
```raku
sub MAIN() {
    # all tokens are True-selectable
    my @tokens = True xx 12;

    my @players = <Player1 Player2>;

    while ( @tokens.elems > 0 ) {
        for @players -> $player {
            my $how-many = ( 1 .. min( 3, @tokens.elems ) ).pick;
            @tokens.pop for 1 .. $how-many;
            say "$player picks $how-many, remaining { @tokens.elems }";

            if @tokens.elems == 0 && $how-many > 0 {
                "Player $player won!".say && exit;
            }
        }

    }

}
}
```
<br/>
<br/>

At every round the `@tokens` array is reduced by a random number between the maximum of `3` and the number of remaining tokens. The player that picks the last token in that round wins and terminates the whole application.
