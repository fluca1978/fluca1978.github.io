---
layout: post
title:  "Perl Weekly Challenge 64: matrix sum and splitting words"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 64: matrix sum and splitting words

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 64](https://perlweeklychallenge.org/blog/perl-weekly-challenge-064/){:target="_blank"}.
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
Our cat Olivia begun to do stairs and can reach the bed. She doesn't walk fine, of course, and still has the extenal metal helper on her hibs. We hope this week she can have the external metal piece removed.

### Eyes

My eyes are getting bad. Unluckily the pressure has raised again, and this means the *xen* is not working fine. I had another injection, last friday, and it was quite hurting this time. I'm really scared about the future, and hardly can think to something else.


<a name="task1"></a>
## PWC 64 - Task 1

The first task was the most difficult one, at least to me.
<br/>Given a matrix, and assumed we can move only in two directions, we need to find the path that provide the lowest sum,
<br/>
I decided to implement this with a specific class:

```perl6
class Node {
    has $.position;
    has $.value;

    has Node $.right is rw;
    has Node $.down  is rw;

    method adjust( @matrix, $m, $n ){
        my $right = $!position + 1;
        my $down  = $!position + $m;
        $!right = Node.new( :position( $right ), :value( @matrix[ $right ] ) ) if $right < $m;
        $!down  = Node.new( :position( $down ), :value( @matrix[ $down ] ) ) if $down <= $n * $m - 1;

        $!right.adjust: @matrix, $m, $n if $!right;
        $!down.adjust: @matrix, $m, $n  if $!down;
    }

    method get-next-min-path(){
        return $!right if $!right && $!down && $!right.value < $!down.value;
        return $!down;
    }
}

```


Every node has a position within the matrix, a value and two particular methods:
- `adjust` that recursively computes the right and down nodes;
- `get-next-min-path` that, for the given node, provides the minimum path. This is suboptimal, because I assume that summing the next min value provides the min sum overall, but I have no idea about how to do an exhaustive sum on all possible values.

<br/>
The final program is therefore:

```perl6
sub MAIN(){
    my @matrix = 1, 2, 3, 4, 5, 6, 7, 8, 9;
    my $m = 3;
    my $n = 3;


    my $start = Node.new( :position( 0 ), :value( @matrix[ 0 ] ) );
    $start.adjust: @matrix, $m, $n;
    my $current-node = $start;
    my $sum = 0;
    my @moves;
    while ( $current-node ) {
        @moves.push: $current-node.value;
        $current-node = $current-node.get-next-min-path;
    }

    say @moves.join( ', ' ) ~ " = " ~ [+] @moves;
}

```

<br/>
that provide an output like the following one:


```perl6
% raku ch-1.p6
1, 2, 3, 6, 9 = 21
```

<a name="task2"></a>
## PWC 64 - Task 2

The second task was a little simpler to me: given a string and an array of words we need to find out if the string can be splitted into the words. The only tricky part appeared to keep the ordering of the words into the string, that does not need explicitly to be that of the array.

```perl6
sub MAIN( Str $S? = 'perlweeklychallenge', @W? = [ "weekly", "challenge", "perl" ] ){

    my $string = $S;
    my @found-words;
    my $redo = True;


    while ( $redo ) {
        $redo = False; # if no one match, skip the next loop
        for @W -> $part {
            # if the current string begins with this token,
            # mark as found and remove from the string
            if $string ~~ / ^ $part / {
                @found-words.push: $part;
                $string ~~ s/ ^ $part //;
                $redo = True;
            }
        }
    }

    # all done
    "In the string $S I found the words { @found-words.join( ',' ) } ".say if @found-words;
    '0'.say if ! @found-words;
}

```

At every pass, if the current word is at the beginning of the string, I mark the loop as succesful and push the word into an ad-hoc array (`@found-words`). After this, I remove the initial word from the string and redo the loop. A small optimization could be to remove the matched word from the array, but I'm not sure the words cannot be duplicated within the very same string.
<br/>
If the loop has been succesfull with at least one word, I have to redo the whole scanning to find out another occurrency, otherwise all has been completed.
<br/>
In the end, I print the array of matching words or `0` as asked by the challenge.
