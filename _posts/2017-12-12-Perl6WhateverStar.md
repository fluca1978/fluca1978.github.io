---
layout: post
title:  "Perl6 whatever star"
author: Luca Ferrari
tags:
- raku
- programming
permalink: /:year/:month/:day/:title.html
---
The *whatever star* is something really special with regard to Perl 6 because it creates a *closure* on its own.

# Perl 6 whatever star

Naming a *whatever star* is quite simple, since it is a simple `*` in the code. Be aware that not all `*` in code are whatever stars, in particular if the `*` appears within a list or a range chances are it is not a *whatever star* rather a limit or boundary.

But what does a whatever star actually do?

## The magic behind whatever star

A **whatever star** is automatically translated to [`Whatever`](https://docs.perl6.org/type/Whatever) class by the compiler, and in turn into a
[`WhateverCode`](https://docs.perl6.org/type/WhateverCode).

In short **a whatever star will produce a closure**.

Let's see this in action: imagine a simple piece of code that requires a closure to upper case a list of words:

```perl
my @words = < blah foo bar baz >;
say @words.map( { $_.uc } );
```

quite clear: on each element of the `@words` array, the `map` method assign `$_` to the current element and the `uc` method is called on it.

How does the whatever star translates applies to the above code? The code became really shorter:

```perl
my @words = < blah foo bar baz >;
say @words.map( *.uc );
```

**the whole closure became `*.uc`**, that can be read as:
1. create a closure with one argument;
2. apply the `uc` method to each element passed to the closure.

## Improving signatures with whatever stars

Every `*` in the block of code for the closure will be considered as a separate argument.
That means that:

```perl
my @words = < blah foo bar baz >;
say @words.map( * ~ *.uc );
```

having two `*` will be as the closure has been defined as:

```perl
say @words.map( -> $a, $b { $a ~ $b.uc } );
```

Please note that **every `*` is managed as a different argument, so in the case you need to backreference a specific argument you cannot use the whatever star**. On the other hand, the code is cleaner and simpler without having the details of the closure explicitly printed out.


## Whatever star for methods-like

Assuming a code block *can* be used as a method with a signature, the following two pieces of code are the same:

```perl
my $func = -> $a, $b { $a + $b };
say $func( 10, 20 );
```

as well as using a couple of stars:

```perl
my $f = * + *;
say $f( 10, 20 );
```


## Summary

When using a *whatever star* be aware of the following simple rule of thumbs:
- it is possible to use *whatever star*s only where a block of code is allowed;
- you don't need the braces to limit the block of code, the whatever star will create a block for you (or better, the compiler will do);
- each `*` is a separate argument on the block signature.
