---
layout: post
title:  "What is precedence dropping call syntax?"
author: Luca Ferrari
tags:
- raku
- programming
permalink: /:year/:month/:day/:title.html
---
If you, like me, are new to Perl 6, the *precedence dropping* syntax can be confusing. I try to explain what I know about.

## What is precedence dropping call syntax?
-----

Perl 6 introduces a lot of syntax changes from its predecessor (or cousin?), and one of it is the so called [**precedence dropping**
in method call](https://docs.perl6.org/language/syntax#Subroutine_calls).

The syntax follows the following rules:
- no need for brackets;
- a comma ```:``` before the argument list;
- all things after the comma are managed as a list.

**Please note that this syntax applies only to *method* calls**, since for subroutine calls it does suffice to avoid parenthesis at all.

So let's see this in action:

```perl
class Foo {
    method foo( *@v ) {
        say @v;
        return 1;
    }
}
````

Now imagine you are going to call as the following:

```perl
my $f = Foo.new;
say $f.foo( 'a', 'b', 'c').^name;
````

which in turn prints:

```perl
[a b c]
Int
````

What happened? Well, what we were expecting: the called the ```foo``` method and then chained the ```name``` on its return value, that is an integer.

Now let's rewrite this with precedence dropping:

```perl
$f.foo: 'a', 'b', 'c'.^name;
````

which produces the following:

```perl
[a b Str]
````

Here the arguments are managed as a whole huge list, so ```name``` is not applied to the method return value but to the last argument.
In other words **precedence dropping means that what is following the method call has the precedence on the call itself, and therefore is evaluated before it is passed to the method itself**.
So the following are equivalent:

```perl
$f.foo: 'a', 'b', 'c'.^name;
$f.foo( 'a', 'b', 'c'.^name );
```

that is what compose the argument list must be evaluated before (i.e., has the precedence) on the method call itself.
Having said that, it is quite clear that the following does not work as expected too:

```perl
$f.foo: ( 'a', 'b', 'c' ).^name; # [List]
````

since again the argument is converted to a whole list, on which is evaluated ```name``` before and then the method is called.

So when is precedence dropping useful? Well, in my opinion when you are hitting the last method call of a chain call you can quite safely apply precedence dropping, and this avoid parenthesis balance problems:

```perl
$f.baz.bar.map.sort.reverse.foo: 'a','b','c';
 #                           ^^^ end of method chain!

$f.bar.baz.sort.foo: 'a', 'b', 'c' .reverse  # wrong!
```
