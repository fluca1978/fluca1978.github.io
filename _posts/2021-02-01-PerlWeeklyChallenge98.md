---
layout: post
title:  "Perl Weekly Challenge 98: reading chars and inserting in array"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 98: reading chars and inserting in array

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 98](https://perlweeklychallenge.org/blog/perl-weekly-challenge-098/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




# My eyes...

I'm waiting and hoping for another consultancy, the next week, by another specialied center.

<a name="task1"></a>
## PWC 98 - Task 1

The first task was quite simple: reading a specific amount of chars from a file, given its filename.
However, there was a little trick: every time a new read was asked for, the file *pinter* need to increase and resume therefore the reading state.
<br/>
The problem asked for the definition of a `readN` function, and I decided to **use an hash to store file handles associated to the filename itself**. With this in mind, it was easy to resume a reading whenever a new read for the same file is asked. Instead of using a global variable, I decide to use a `state` variable, that is initialized only the first time the function is called, and therefore can be used later on.
<br/>
The resulting function is therefore as simple as:

<br/>
<br/>
```raku
sub readN( Str $file-name, 
           Int $how-many-chars where { $how-many-chars >= 1 } ) {
    state %handles = ();
    %handles{ $file-name } //= $file-name.IO.open;
    return %handles{ $file-name }.readchars: $how-many-chars;
}
```
<br/>

If a file stream is found in the `%handles` hash, the stream is used and `readchars` is called. Otherwise, the hash is initialized by means of `//=` with a new value.

<br/>
Using the function therefore is as simple as:


<br/>
<br/>
```raku
sub MAIN( Str $file-name ) {
    say readN( $file-name, 4 );
    say readN( $file-name, 4 );
    say readN( '/home/luca/.zshrc', 10 );
}
```
<br/>
<br/>


<a name="task2"></a>
## PWC 98 - Task 2

In the second task there was the need to search for a value in an *ordered array of integers*: if the value is found, the index has to be printed, otherwise the array has to be modified and the new element has to be inserted preserving the ordering.
<br/>
I decided to implement the two branches by means of `given`. The easiest case is the one where there is a direct match: `.grep` can be used to find out a match, and we are interested in the very `.first` match found.
In the other case, I need to search for the `.first` index that has a value greater or equal of the searched one, then a new array is composed by splitting the original one in two parts and by inserting the new value in the middle. Therefore it resolves as:

<br/>
<br/>
```raku
sub MAIN( *@values where { @values.grep: * ~~ Int } ) {
    my @N = @values[0 .. *-2 ];
    my $N = @values[ *-1 ];

    # get the index if the element is there
    given @N.grep( $N, :k ).first { .say && exit if $_ }

    # if here the key is not there, let's see where to insert
    given @N.grep( { $_ >= $N }, :k ).first {
        @N = |@N[ 0 .. $_ - 1 ], $N, |@N[ $_ .. * ];
        .say && exit;
    }

}

```
<br/>
<br/>

Both the `given` branches print the index and exit, so there is no particular added value in modifying the array.
Please note that, in the case a new value is inserted, the arrays slices are flatted by means of the `|` operator.
<br/>
Thanks to the `:k` adverb of `grep` it is trivial to find out the index to operate on.
