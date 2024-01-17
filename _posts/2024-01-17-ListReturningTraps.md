---
layout: post
title:  "Small traps in Perl functions that return lists"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
Sometimes it is not so simple to catch returning lists.

# Small traps in Perl functions that return lists

I often find some code that works as follows: given a list of items, there is a function that provides a list of *unique* items, and another function that counts how many unique items are there.

The code looks like the following:

<br/>
<br/>
```perl
#!perl
use v5.38;

sub unique_list {
    my $unique = {};
    $unique->{ $_ }++ for ( @_ );
    return sort keys $unique->%*;
}

sub unique_count {
    return scalar( unique_list( @_ ) ) // 0;
}

```
<br/>
<br/>


The problem is within `unique_count`: the function does not "store" the result of `unique_list` into a list, but passes it to `scalar`, that results in a zero or undef value. Just think at what happens if the `unique_list` returns a single value.

Assuming the following code:

<br/>
<br/>
```perl
my @animals = qw/ cat dog fish cat dog bird /;

say unique_list @animals;
say unique_count( @animals );

```
<br/>
<br/>

The `unique_count` will always return zero, even if `unique_list` is working as expected.

<br/>
It is Perl!
<br/>

There are several solutions to this, one could be to assign the result of `unique_list` to a list:

<br/>
<br/>
```perl
sub unique_count {
    my @unique = unique_list( @_ );
    scalar( @unique );
}

```
<br/>
<br/>

or use an array reference when dealing with `unique_list`:

<br/>
<br/>
```perl
sub unique_list {
    my $unique = {};
    $unique->{ $_ }++ for ( @_ );
    return [ sort keys $unique->%* ];
}

sub unique_count {
    return scalar( unique_list( @_ )->@* ) // 0;
}

```
<br/>
<br/>

Clearly, the latter solution means that users of `unique_list` has to de-reference the returned value before it can be used:

<br/>
<br/>
```perl
say unique_list( @animals )->@*;
say unique_count( @animals );

```
<br/>
<br/>

The solution to adopt is pretty much a matter of taste, I tend to prefer the array ref solution because it makes much more clear that I have to derefence an array, so there are less chance I'm going to forget to use a list in the counting-zone.
