---
layout: post
title:  "Perl Weekly Challenge 45: encoded messages and self-source-code-printing"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 45: encoded messages and self-source-code-printing

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 45](https://perlweeklychallenge.org/blog/perl-weekly-challenge-045/){:target="_blank"}.

<br/>
My official solutions are available [on the GitHub repository](https://github.com/fluca1978/perlweeklychallenge-club/tree/master/challenge-045/luca-ferrari){:target="_blank"}.

## PWC 45 - Task 1

The task asks to get a message, remove all the spaces and distribute the message across an eight column matrix. Finally, print the message by columns instead of rows. This is called *the squate secret code*.
<br/>
To better understand, let's consider the following example:

```shell
% perl6 ch-1.p6 --message="The quick brown fox jumps over the lazy dog"
Your original message is
The quick brown fox jumps over the lazy dog
and encoded results:

  t h e q u i c k
  b r o w n f o x
  j u m p s o v e
  r t h e l a z y
  d o g

  that leads to

 tbjrdhrutoeomhgqwpeunslifoacovzkxey
```

<br/>
First of all, I need to get a matrix with all non spaces chars:

```perl6
for $message.lc.comb( / \w / )  {
    @matrix[ $row ].push: $_ if @matrix[ $row ].elems < $columns;
    $row++ if @matrix[ $row ].elems == $columns;
}
```

The `comb` method extracts only the letters (non-spaces) from the string `$message`, then I `push` every single letter into `@matrix[ $row ]`, that is an array of array (indexed by `$row`) up until `$columns` (i.e., 8 columns). Once I hit the `$columns` (i.e., 8) upper limit, I change the matrix row.
<br/>
Now it's time to print the matrix by columns:

```perl6
for 0 .. $columns -> $start {
    ( @matrix[ $_ ][ $start ] // '' ).print for 0 .. $row;
}
```

I loop over the `$columns` first, and then I do print the current position between all the matrix rows (i.e., between `0 .. $row`, with the latter being increased by the preceeding loop).
Please note that the matrix could have the last line not completed, so I print a char or an empty char if nothing is defined in that matrix position.

### A Smaller Solution

I was not satisfied about the above solution, and after a while I remembered that strange method that I seldom use: `rotor`. This method splits a list into sublists of the specified size, and with the `:partial` adverb, can include also lists that are not complete. Therefore, the whole first loop can be reduced to:

```perl6
my @matrix = $message
               .lc
               .comb( /\w/ )
               .rotor: 8, :partial;
```

that produces the very same `@matrix` as the loop before, and moreover, the `@matrix.elems` corresponds to the number of rows the matrix has. From the above, the printing loop becomes:

```perl6
for 0 .. $columns -> $start {
    ( @matrix[ $_ ][ $start ] // '' ).print for 0 .. @matrix.elems;
}
```

## PWC 45 - Task 2

This has been very easy, a single line script: provide a program that prints its own source code.
Since in Raku, the dynamic variable `$*PROGRAM` olds an `IO` object, I can simply loop over the `lines` and print them out.

```perl6
  .say for $*PROGRAM.lines;
```

Either this has been too simple, or I didn't understand the task assignment!
