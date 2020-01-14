---
layout: post
title:  "Perl5 -> Perl6: String interpolation"
author: Luca Ferrari
tags:
- raku
- perl
- programming
permalink: /:year/:month/:day/:title.html
---
Allow me to remark a few differences in how Perl 6 does variables substitution within strings, and how to use the *zen slice*.

# Perl5 -> Perl6: String interpolation

## Perl 5 (simple) interpolation

Perl 5 allows for *string interpolation* when using double quote strings, so that you can place directly a variable within the string and it will get interpoalted and substituted by its value. Of course, if the variable is something more than a *base type*, e.g., a reference to an object, it will be printed out as a *stringyfied* version such as `rafType<Address>`.

In other words, the following simple code snippet:

```perl
my $scalar = 'Hello World!';
my @array  = ( 'Hello', 'World', '!' );
my %hash   = ( Hello => 'World!' );

say "Scalar is [$scalar]";
say "Array is  [@array]";
say "Hash is   [%hash]";
```

will produce the following output:

```shell
Scalar is [Hello World!]
Array is  [Hello World !]
Hash is   [%hash]
```

Therefore, with the very exception of hashes, Perl 5 is smart enough to interpolate scalars and arrays within a string.
Now, why are not hashes interpolated? For a lot of good reasons, including the fact that the `%` sign denotes an escape sequence in many places.
However, it is possible to evaluate an hash in *list context*, in such case it will be interpolated in the obvious way:

```perl
my %hash   = ( Hello => 'World!' );
say "Hash is   [" , %hash , "]";
```

will produce the following:

```perl
Hash is   [HelloWorld!]
```


## Perl 6 (simple) interpolation

How does Perl 6 deal with the very same code?
The following piece of code represents the same program for a quick comparison:

```perl
my $scalar = 'Hello World!';
my @array  = 'Hello', 'world!';
my %hash   = Hello => 'World!';

say "Scalar is [$scalar]";
say "Array is  [@array]";
say "Hash is   [%hash]";
say "Hash is   [" , %hash , "]";
```

and the result is as follows:

```shell
Scalar is [Hello World!]
Array is  [@array]
Hash is   [%hash]
Hash is   [{Hello => World!}]
```

As you can see **Perl 6 does not automagically interpolate arrays and hashes**, or better, it does not the way Perl 5 does.
The reason is that Perl 6 requires to know the **subscript** it is going to interpolate, and therefore just naming a variable does not produce a subscript, so the only chance is to **subscript all the content** in what is called the **[zan slice](https://docs.perl6.org/language/subscripts#Zen_slices)**, **a slice without any index** (something like *"take it all"*):

```perl
say "Scalar is [$scalar]";
say "Array is  [@array[]]";
say "Hash is   [%hash{}]";
```

and the above code now produces:

```shell
Scalar is [Hello World!]
Array is  [Hello world!]
Hash is   [Hello        World!]
```


## Zen or Star?

It is worth noting that **a zen slice is different from the _whatever star_ one**.
From a syntactic point of view a zen slice is specified without including any index or list in the subscript, while a whatever start does include a `*` as subscript.
Now, while the two may seem similar, the **whatever star returns the full list of values as if all the keys have been specified**.
This is not different within array interpolation, while it is really different within hashes:

```perl
say "Array is  [@array[*]]";
say "Hash is   [%hash{*}]";
```

produces

```shell
Array is  [Hello world!]
Hash is   [World!]
```

As you can see, the array has been entirely stringified (since it is like an array has no special key to be interpolated), while the hash is interpolated only with its values (since it was like all the keys were asked).
