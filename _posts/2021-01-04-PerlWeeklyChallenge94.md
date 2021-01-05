---
layout: post
title:  "Perl Weekly Challenge 94: I'M (nearly) BACK!"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 94: I'M (nearly) BACK!

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 94](https://perlweeklychallenge.org/blog/perl-weekly-challenge-094/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



# I'M (nearly) BACK

My last PWC contribution was [number 76](https://fluca1978.github.io/2020/08/31/PerlWeeklyChallenge76.html){:target="_blank"}.
<br/>
Yes, I was **away from keyboard for almost four months** due to an eye surgery, [trabeculectomy](https://en.wikipedia.org/wiki/Trabeculectomy){:target="_blank"}, that has done good results on the eye pressure (the main aim), but quite bad results on the eyesight.
<br/>
Therefore, I'm not really happy with that, and I'm trying to figure out how to deal with my **new (downgraded) life**.
<br/>
As a way to force my brain to stay focused on something different, I decided to start over the *Perl Weekly Challenge*.
<br/>
This is also why my [blog activity has dropped in the last quarter of 2020](https://fluca1978.github.io/stats/).{:target="_blank"}.

<a name="task1"></a>
## PWC 94 - Task 1

The first task was about getting out *anagrams* by a set of words, i.e., different words written in different ways but with the very same letters.
I wrote a `get-anagrams` function that accepts the array of string and iterate on every string of the array.
For every string found in the array, from the beginning to the end, I compute all the possible permutations of the letters that compose such string, and then try to see if there is another word that matches the resulting anagram. If there is a word, such word is added to a `@results` temporary array, that in turn is materialized into the global `@results` array within `MAIN`.
<br/>
Since the string array is iterated from the very beginning to the end, there is no possibility there is a repetition of the same anagrams.
<br/>
<br/>
```raku
sub get-anagrams( @S where { @S.elems > 1 } ) {
    my @results;

    # cycle thru the words using ordering
    # so that the same anagram is not
    # backward referenced
    for 0 ..^ @S -> $outer-index {

        # use a temporary array to store these anagrams
        # and initialize it with the current word
        my @anagrams = @S[ $outer-index ];
        for @anagrams[ 0 ].comb.permutations {
            my $current = .join;
            # push this word if it is not the same as the current one,
            # it is within @S and is not already into @results
            @anagrams.push: $current if ( @S[ $outer-index .. *-1 ].grep( $current )
                                          && ! @anagrams.grep( $current )
                                          && ! @results.List.flat.grep( $current ) );
        }

        # if there is more than one word (the first one is the current)
        # then there is at least an anagram, so push to the final results
        @results.push: [ @anagrams ] if ( @anagrams.elems > 1 );
    }

    return @results;
}
```
<br/>
<br/>

It is quite simple then to produce a `MAIN`:

<br/>
<br/>
```raku
multi sub MAIN( *@S ) {
    my @results = @S.elems > 1 ??  get-anagrams( @S ) !! @S;
    say "Valid anagrams: { @results.join(' - ') }";
}

multi sub MAIN(){
    my Str @S = <opt bat saw tab pot top was>;
    my @results = get-anagrams( @S );
    say "Valid anagrams: { @results.join(' - ') }";
}
```
<br/>
<br/>

The special case for `@S` with a single string begin returned as the anagram of itself is a special case requested by the task itself.
<br/>
Invoking the program produces the result as follows:

<br/>
<br/>
```shell
% raku ch-1.p6        
Valid anagrams: opt pot top - bat tab - saw was
```
<br/>
<br/>

There is room for improvements:
- ensure that all searched words have the very same length (it is not clear to me if this should be a requirement of `@S` or must be ensured at run-time);
- `grep` against different cases (upper or lower case).



<a name="task2"></a>
## PWC 94 - Task 2

The second task was not too clear to me: it is required to print out a tree structure as a linked list.
<br/>
My doubts are about:
- do we have to produce a list and then print the list or just print the tree as a list?
- is the tree to be printed left to right when dealing with nodes?
<br/>
I assume no list has to be created and that the tree must be printed from left to right when inspecting the tree.
<br/>
First of all I created a tree-like structure with very hand `Node` class:

<br/>
<br/>
```raku
class Node {
    has Int $.value  is rw;
    has Node $.left  is rw;
    has Node $.right is rw;


    method me-recursive() {
        my $str = self.me;
        $str ~= ' -> ' ~ $!left.me-recursive  if $!left;
        $str ~= ' -> ' ~ $!right.me-recursive if $!right;
        $str;
    }

    method me() { $!value; }

}

```
<br/>
<br/>

All the magic happens in the `me-recursive` method:
- it does print the current node value;
- appends the arrow and the value of the `$!left` node, if any;
- appends the arrow and the value of the `$!right` node, if any.

<br/>
Since every call to `me-recursive` proceed in recursion on the left node first, the tree is printed from left to right until no children have been found, than the traversing starts over the right direction.

<br/>
The end result is the one asked from the challenge:
<br/>
<br/>
```shell
% raku ch-2.p6        
1 -> 2 -> 4 -> 5 -> 6 -> 7 -> 3
```
<br/>
<br/>

Again, I'm not sure I've fully understood this challenge.
