---
layout: post
title:  "Perl Weekly Challenge 110: Mangling Text Files"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 110: Mangling Text Files

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 110](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0110/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 110 - Task 1

In the first task I have to select a few valid phone numbers depending on some specifications.
<br/>
This task could be easily solved with a bunch of regular expressions, used to validate every line in input. I decided to place the regular expressions into an array, so that I can use a *junction* to quickly test on a single line if the input matches *any* of the regular expression, and if so, I do print the matching line on standard output.
<br/>
The code is therefore the following:

<br/>
<br/>
```raku
sub MAIN( Str $file-name = 'phone.txt' ) {
    my @regexps = rx / ^ \s* <[+]> \d ** 2 \s+ \d ** 10 $ /
                , rx / ^ \s* <[(]> \d ** 2 <[)]> \s+ \d ** 10 $ /
                , rx / ^ \s* \d ** 4 \s+ \d ** 10 $ /;

    $_.say if $_ ~~ any @regexps for $file-name.IO.lines;
```
<br/>
<br/>

As you can see, the main idea is to iterate on every file lines, than smart match against *any* of the defined regular expressions.



<a name="task2"></a>
## PWC 110 - Task 2
The second task was about transposing the content of a *CSV-like* text file.
<br/>
In order to transpose every line, I decided to put the lines into an array, than to print out the array *re-mapped* in a columnar way.
<br/>
My first implementation was quite trivial:


<br/>
<br/>
```raku
sub MAIN( Str $file-name = 'people.txt' ) {

    my @content;
    @content.push: .split( ',' ) for $file-name.IO.lines;

    for 0 ..^ @content[ 0 ].elems -> $column {
        my @row.push:  @content[ $_ ][ $column ] for 0 ..^ @content.elems;
        @row.join( ',' ).say;
    }
}
```
<br/>
<br/>

The main part is the `for` loop, where I iterate on every column (assuming `@content[ 0 ]` contains the exact number of columns that are in all the other rows), and I `push` the content of every row at that column in another array named `@row`, that is then printed out.
<br/>
But this is too much code, so I used the `map` function to do what the `for` loop does:

<br/>
<br/>
```raku
sub MAIN( Str $file-name = 'people.txt' ) {

    my @content;
    @content.push: .split( ',' ) for $file-name.IO.lines;

    my @transposed.push: @content.map: *[ $_ ] for 0 ..^ @content[ 0 ].elems;
    $_.join( ',' ).say for @transposed;

}
```
<br/>
<br/>

The `map` iterates on every line, and on every line it does extract the current column by means of an inner `for` loop, then the result is placed into `@transposed` array that is then printed one line at a time.
