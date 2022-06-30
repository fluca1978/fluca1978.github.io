---
layout: post
title:  "The importance of postfix operators in Perl"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
An important thing I discovered on postfix operators in Perl!

# The importance of postfix operators in Perl

As everyone knows, Perl (and hence Raku) has a set of postfix operators, like `if`, `for`, `while`, `unless`.
<br/>
In my early age of Perl programming I was not using them, since I felt *strange* in adopting constructs that not any other language I used had.
<br/>
However, gradually, I started using them mainly because of how much they improve the code readibility.
<br/>
But there's another reason why they are important and you should adopt them: **they tend to be faster**!
Let's write a simple script to prove the above statement:


<br/>
<br/>

``` perl
use v5.20;
use DateTime;
my $x = 0;
my $limit = 10_000_000_000;
my $start = DateTime->now;

++$x  ** $_  for ( 1 .. $limit );

my $end =  DateTime->now;

say "Postfix: " . ( $end->epoch - $start->epoch );

$x = 0;
$start = DateTime->now;
for ( 1 .. $limit ) {
 ++$x ** $_;
}

$end = DateTime->now;
say "Prefix: " . ( $end->epoch - $start->epoch );

```
<br/>
<br/>


## TL;DR Show me the results!

Before digging into the above simple script, let's see what it outputs and then reason about that:


<br/>
<br/>

``` perl
% perl tmp/test.pl
Postfix: 694
Prefix: 716
```
<br/>
<br/>

As you can see **the *postfix* form of the `for` loop is almost 20 seconds faster** than the equivalent prefix form.
<br/>
The script does exactly the same thing in both the `for` versions: it does *a lot of iterations* and computes something. Why? Because, clearly, you cannot measure the difference between something small!
<br/>
The script has been executed on Perl `5.34.1`, but similar results can be obtained on other versions.


## Why is the postfix version faster?

Glad you asked!
<br/>
The point is that **postfix version does not create a new scope**: the *prefix* `for` loop has a set of curlies `for { ... }`, while the postifx does not. Every time Perl encounters a set of curliy braces, it has to create a new variable scope. And that is what every prgorammer usually wants, right?
<br/>
However, **creating a new variable scope has a cost**, and even if small, such cost is going to impact your application. Therefore, if you don't need the scope, use postfix operators!


## Conclusions

Postfix operators are great, not only because they greatly improve code readability, but also because they can spee-up the execution when no new scope is needed.
<br/>
This does not mean that you need to use postfix oeprators whenever you can, after all we are talking about **small performance improvements** that can clash with code safety (the scope is protecting you from mistakes somehow), but be assured that when you do, the code will run faster!
