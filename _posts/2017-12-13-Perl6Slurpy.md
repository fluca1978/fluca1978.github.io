---
layout: post
title:  "Perl5 -> Perl 6: slurpy things"
author: Luca Ferrari
tags:
- raku
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
Perl 6 does not use by default the *slurpy* behavior of his older brother Perl 5.

# Perl5 -> Perl 6: slurpy things

Perl 5 is famous for having a *slurpy* nature, especially when dealing with method arguments. In Perl 6 this behavior has changed, so the developer must be aware and instrument the compiler for when it should expect a slurpy argument.

## Being *Slurpy*: the Perl 5 way

Perl 5 passes all arguments to a function or a method via the topic array `@_`.
When I was approaching Perl 5 coming from other *typed* languages (like C, Java, etc.) I thought this behavior was horrible. Doing more and more Perl 5 development, I found it really smart and compact: **you name parameters only when (and where) you need them**, otherwise just consume them as a flat list.

This slurpy behavior means that **you have to use references to structured data as arguments** or you will not be able to distinguish parameters within the same **flat list**.

With that in mind, the following program illustrates the Perl 5 slurpy behavior:

```perl
use v5.20;

sub hello {
    my ( @args ) = @_;
    say 'hello got ' . scalar @args . ' arguments';
    say join ' -> ', @args;
}

sub hello_2 {
    say 'hello_2 got '. scalar @_ . ' arguments';
    say join ' ->> ', @{ $_ } for @_;

}

hello 1, 2, 3;
hello( (1, 2), qw( a b ) );

hello_2( [1, 2], ['a', 'b'] );
```

producing the following output

```shell
hello got 3 arguments
1 -> 2 -> 3
hello got 4 arguments
1 -> 2 -> a -> b
hello_2 got 2 arguments
1 ->> 2
a ->> b
```

In order to recap it is possible to see that:
- `hello` got a flat list even when two lists are passed as arguments, what the function "sees" is a single list made up by all the elements;
- `hello_2` again sees a single flat list, this time made up by two references to arrays and therefore it is able to elaborate such references in order to get the array identities.


## Being *not slurpy*: the Perl 6 way

In Perl 6 you declare exactly what type of arguments your method is going to receive, and this will prevent the generation of the flat *catch-all* list.
That means that the above program simply becomes:

```perl
sub hello( @args ) {
    @args.join( ' -> ' ).say;
}

sub hello_2( @arg1, @arg2 ) {
    @arg1.join( ' ->> ' ).say;
    @arg2.join( ' ->> ' ).say;
}

hello <1, 2, 3>;
hello_2 (1,2,3), ('a', 'b');
```


producing the output

```shell
1, -> 2, -> 3
1 ->> 2 ->> 3
a ->> b
```

What happens if we pass two arrays to the `hello` method that accepts only one?

```perl
hello (1,2,3), ('a', 'b');
```

the complier will claim the mistake since it is not able to understand what is going on:

```shell
Too many positionals passed; expected 1 argument but got 2
  in sub hello at test.p6 line 4
  in block <unit> at test.p6 line 14
```

So Perl 6 is saying quite loudly we declared the method in a specific way, and we cannot call in a more general way.

## Being *slurpy*, the Perl 6 way

It is possible to instrument Perl 6 to accept a slurpy parameter, of course not a scalar, using a `*` in front of the parameter type.
Once Perl 6 encounters a slurpy parameter, nothing else will be considered (of course), so **slurpy parameters must be at the end of the declaration**:

```perl
sub hello( *@args ) {
    @args.elems.say;
    @args.join( ' -> ' ).say;
}

hello (1,2,3), ('a', 'b');
```

that produces the following output:

```shell
1 -> 2 -> 3 -> a -> b
```

It is important to note that all parameters are printed within the same line, that is within the same `say` context, as the array is effectively a flat list of arguments.

But Perl 6 allows also for another slurpy mode: the `**` slurpy mode.
Modifying the function declaration the program changes its behavior:

```perl
sub hello( **@args ) {
    @args.elems.say;
    @args.join( ' -> ' ).say;
}

hello (1,2,3), ('a', 'b');
```

producing the following output:

```shell
2
1 2 3 -> a b
```

What has changed? With the double-star the argument has kept its slurpy behavior, but it has also preserved the identifies of the single lists within the array.
In other words __`*@` provides a slurpy flat behavior, while `**@` provides a slurpy structured behavior__.
