---
layout: post
title:  "Perl Weekly Challenge 63: words and rotations"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 63: words and rotations

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 63](https://perlweeklychallenge.org/blog/perl-weekly-challenge-063/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation here in Italy is slowly coming back to normal: several places have openened the last week and while we are still forced to stay at home as much as possible, we can move a little more freely.
<br/>
I have visited my mom a couple of other times in the last week, and she came to visit me too.
<br/>
I've worked from office a couple of times, but still I do the main activity from home.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine**, at least for the time I spend in front of a Perl script!


### Olivia
Me and my wife removed the collar yesterday, and the cat is walking around the house but we don't allow her to take stairs and jump on the sofa.
<br/>
She still has the external metal helper on her hibs, we need to wait a couple of week before we can decide to remove it by aoher surgery. So far, she is not walking very well, but at least she is walking.


<a name="task1"></a>
## PWC 63 - Task 1

The first task was about writing a function that allows to match a word into a sentence, given a specific regular expresison.
Apparently this was easy, but it took me two trials to get it right.
<br/>
The requested function can be written as:

```perl6
sub last_word( $string, $regexp ){
    my $last-word = Nil;
    for $string.split( " " ) {
        $last-word =  $_ if $_ ~~ $regexp;
    }

    return $last-word;
}
```

The idea is that the `$last-word` assumes every single word from the `split` that matches the regexp. Therefore, at the end of the loop, the `$last-word` contains either the last word in the sentence or `Nil`.
<br/>
Another, sligtly smarter approach is about reversing the words produced by `split`, so that the first match passes:


```perl6
sub last_word_shorter( $string, $regexp ){
    for $string.split( " " ).reverse {
        return $_ if $_ ~~ $regexp;
    }
}
```


The `MAIN` is used only for testing purposes:

```perl6
sub MAIN(){
    say last_word('  hello world',                rx/<[ea]>l/);      # 'hello'
    say last_word("Don't match too much, Chet!",  rx:i/ch.t/);      # 'Chet!'
    say last_word("spaces in regexp won't match", rx:s/ in re / );      #  undef
    say last_word( join(' ', 1..1e6),             rx/ ^ (3.*?) ** 3 / ); # '399933'

    say last_word_shorter('  hello world',                rx/<[ea]>l/);      # 'hello'
    say last_word_shorter("Don't match too much, Chet!",  rx:i/ch.t/);      # 'Chet!'
    say last_word_shorter("spaces in regexp won't match", rx:s/ in re / );      #  undef
    say last_word_shorter( join(' ', 1..1e6),             rx/ ^ (3.*?) ** 3 / ); # '399933'
}
```



<a name="task2"></a>
## PWC 63 - Task 2

The second task was about doing a progressive transformation on a string, depending by the iteration number. Essentlially, at every iteration a specific number of chars have to be moved from the beginning to the end, and the cycle repeats until the string has rotated all chars.
I decided to implement as follows:


```perl6
sub rotate( $string where { $string.split( '', :skip-empty ).chars > 1 }, $verbose = False ){
    my $step = 1;
    my @chars  = $string.split( '', :skip-empty );


    while ( $step == 1 || $string ne @chars.join( '' ) ) {
        say "Rotation $step the string is: {@chars}" if $verbose;

        # move the first character to the end
        @chars.push: @chars.shift for 0 ..^ ( $step % $string.chars );
        

        say "After step $step the string is: { @chars }" if $verbose;

        $step++;
    }

    return $step - 1;
}
```

First of all, the function accepts the `$string` (with the restriction it has to be longer than a single char) and a `$verbose` flag.
Then the function splits the string into an array of characters, and on every iteration pushes to the end the first char of the string depending on the result of `$step % $string.chars`. If the string returns to its original value, the `while` stops and the function returns the number of iterations (minus 1).
