---
layout: post
title:  "Adverb pairs in Perl 6"
author: Luca Ferrari
tags:
- perl6
- programming
permalink: /:year/:month/:day/:title.html
---
A `Pair` is a *key-value* couple, often used with hashes. Perl 6 does support several ways to build pairs, the most common one is with the *fat comma* `=>` operator. But also adverbs can be used to construct `Pair`s easily.

# Pairs

`Pair`s are usually built with the *fat-comma* operator:

```perl
( name => 'Luca', surname => 'Ferrari' )[0].say
# name => Luca
( name => 'Luca', surname => 'Ferrari' )[0].^name.say
# Pair
```

## Using adverb notation to create `Pair`s

A `Pair` can be built with the adverb notation, using `:`, in several ways.

### Explicit key and value

In this form the `Pair` assume the form `:key(value)` or, with automatical quotes on values, the form `:key<value>`:

```perl
:name('Luca').say;
# name => Luca
:name('Luca').^name.say;
# Pair

:name<Luca>.say;
# name => Luca
:name<Luca>.^name.say;
# Pair
```
### Implicit value

In this form a `Pair` is created by simply placing a colon in front of a variable sigil. The `Pair` will use the *name* of the variable as key and the value of the variable as *value* in the pair:

```perl
my $name = 'Luca'; # same as $name = <Luca>
# Luca
:$name.say;
# name => Luca
:$name.^name.say;
# Pair
```


### Implicit boolean

In the case no sigil is specified, the name of the adverb is used as a key, the boolean value `True` is assigned (or false if a negation is used):

```perl
:male.say;
# male => True
:male.^name.say;
# Pair

:!male.say;
# male => False
!male.^name.say;
# Pair
```

### Implicit value within key

In the special case of a number, it is possible to put the number in front of the key name and the `Pair` will be defined with the number as value. This is, in my opinion, a bad approach because it can make things quickly unreadable:

```perl
:39Age.say
# Age => 39
:39Age.^name.say;
# Pair
```
