---
layout: post
title:  "What is block syntax?"
author: Luca Ferrari
tags:
- raku
- programming
permalink: /:year/:month/:day/:title.html
---
If you, like me, are new to Perl 6, the *block* syntax can be confusing. I try to explain what I know about.

## What is block syntax?
-----

The documentation states that each time you see an arrow ```->``` you are in front of a [block syntax](https://docs.perl6.org/language/functions#index-entry-syntax_-%3E-Blocks_and_Lambdas), with the definition of a signature.
This applies to iterations and methods that accept code blocks (in Perl 6 a [Block is a class by its own](https://docs.perl6.org/type/Block), not surprisingly).

But what does it mean? Allow me to try to explain with a simple example: transforming an array thru ```map```, that as you know, accepts a block.
Let's start simple:

```perl
my @v = 1, 2, 3, 'a', 'b', 'c';
say @v.map: { $_ ~ '-mapped' };
```

which outputs a list of elemnts with the postfix "mapped" string:

```
(1-mapped 2-mapped 3-mapped a-mapped b-mapped c-mapped)xo
```

There is nothing really interesting here: it does resemble (of course) the Perl 5 way of doing things. ```map``` exploits the topic variable
```$_``` within the block of code.
The above piece of code is totally equivalent to the same, that is its explicit form:

```perl
my @v = 1, 2, 3, 'a', 'b', 'c';
say @v.map: -> $_ { $_ ~ '-mapped' };
```

Here we introduce a signature (thru the ```->```) before the argument list. What it does mean, for short, is that the following piece of code
will work with the signature as input, in a way similar to the definition of a method prototype.
So for instance, you can decide to use another variable instead of the topic one:

```perl
my @v = 1, 2, 3, 'a', 'b', 'c';
say @v.map: -> $current { $current ~ '-mapped' };
```

In the above the ```$current``` variable is used as "input" for the block, and that is the way the block syntax work.
But, of course, this opens a wide range of new ways of exploiting things. For instance, if we want to concatenate each pair of elements we can use another signature:

```perl
my @v = 1, 2, 3, 'a', 'b', 'c';
say @v.map: -> $one, $two { $one ~ '-mapped-' ~ $two };
```

that produces the following output:

```
(1-mapped-2 3-mapped-a b-mapped-c)
```

So, with signatures we can *manipulate* the way blocks receive input, in a similar fashion to subroutine prototypes. And, similarly to
subroutine prototypes, we can specify also default values:

```perl
my @v = 1, 2, 3;
say @v.map: -> $one, $two = 4 { $one ~ '-mapped-' ~ $two };
```

that produces

```
(1-mapped-2 3-mapped-4)
```

What happens is that at the first iteration two elements are popped from ```@v```, and placed into ```$one``` and ```$two```. At the second
iteration only one element is left to be popped from ```@v```, so only ```$one``` can be populated and therefore ```$two``` assumes the default value specified in the signature.

At glance, signatures applied to blocks can be confusing, especially for the extra arrow, but in the long term being aware of them allows for a more simple code and reduce of complexity.
