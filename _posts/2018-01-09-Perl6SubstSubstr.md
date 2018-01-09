---
layout: post
title:  "Perl 6 subst vs substr"
author: Luca Ferrari
tags:
- perl6

permalink: /:year/:month/:day/:title.html
---

There is a momentum around a possible rename/deprecation of a Perl 6 method to avoid typing clashes.

# `subst` vs `substr`

So what is this thing about? As discussed [mainly here](https://github.com/perl6/6.d-prep/issues/3#issuecomment-355856478) and [here](https://github.com/rakudo/rakudo/issues/1314#issuecomment-354563216) there is a possible clash between two different methods in Perl 6 that have been abbreviated in a very similar way:
- `subst`, short for *substitute* that performs a `s//` against a `Str`;
- `substr`, short for *substring*, that extract data from a `Str`.

The latter, `substr` is a *routine*, that is a method that can be called as an operator against an object:

```perl
substr 'hello', 0, 1; # h
'hello'.substr: 0, 1; # h
```

The former, `subst` is a *method*, i.e., it is not possible to call it outside of an object instance:

```perl
> subst 'hello', 'h', 'H';
===SORRY!=== Error while compiling:
Undeclared routine:
    subst used at line 1. Did you mean 'substr'?

> 'hello'.subst: 'h', 'H';
# Hello
```


The problem, clearly, arises when you try to use any of the above as *methods*, that is against an object instance:

```perl
# mispelled 'substr' instead of 'subst'
> 'hello'.substr: 'h', 'H';
Earlier failure:
 Cannot convert string to number: base-10 number must begin with valid digits or '.' in '⏏h' (indicated by ⏏)
  in block <unit> at <unknown file> line 1

Final error:
 Type check failed in assignment to $from; expected Int but got Failure (Failure.new(exception...)
  in block <unit> at <unknown file> line 1

# mispelled 'subst'  instead of 'substr'
> 'hello'.subst: 0, 1;
hello
```

Since the two methods differ only on the final 'r', it is quite easy to mispell them and got a compilation error or a wrong result. In fact, while `substr` cannot accept strings as input (unless they are number stringitied!), it will cause a compilation error. On the other side, ~`subst` does accept numbers as input and in the better case it does nothing (not finding any occurence of the first number).

You get the point: the situation can quickly become a mess!

That's why there's some thinking effort about if, how and when to rename any of the method to another name and, possibly, deprecate the old name.
My personal idea, as I wrote in the [discussion comments](https://github.com/perl6/6.d-prep/issues/3#issuecomment-355856478) is to:
- avoid deprecating any method, since they have been potentially used in many places so far;
- provide a long method name for both keeping the short names in place, so that every developer can choose the style he likes the most.

Being `substr` a routine, particular emphasis should be kept to that, and therefore I strongly recommend to not change at all its name. On the other hand, being `subst` a method, it is possible to alias it and suggest developers to use the right name to avoid bugs due to the usge of the wrong method.

It is not a simple task, and it surely opens a path for other similar cases to be managed in the future, and therefore it does require a particular care. Is this a possible API mistake in the Perl 6 language? I'm not so sure, but it is a surely possible annoying capability.
