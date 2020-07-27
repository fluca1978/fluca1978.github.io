---
layout: post
title:  "Two of my most frequent syntax mistakes in Raku"
author: Luca Ferrari
tags:
- raku
permalink: /:year/:month/:day/:title.html
---
Raku has awesome error messages!

# Two of my most frequent syntax mistakes in Raku

Raku is, together with Perl, my favourite programming language. Unluckily, I cannot use in my day-to-day activity and, most notably, I cannot use for *every* activity. This means I have to do some stuff in Raky, some in Java, PHP, Perl, and who knows whatever else.
<br/>
<br/>
It happens therefore, sometime, that I confuse language syntaxes.
<br/>
What makes Raku great is that the compiler provides very detailed error messages, sometimes making me laugh at how stupid I am for the error I've done.
<br/>
The following are the couple of mistakes I do the most when switching to Raku.


## Unsupported use of C++ constructor syntax

Raku, as other *sane* OOP languages, use methods to do everything, including constructing objects. This means there is no special operator `new`. Actually, you can find this also in Perl, but not in other OOP languages like Java and PHP.
<br/>
What I sometime do is:

<br/>
<br/>
```raku
class Foo {
    has $.bar;
}

my $foo = new Foo( bar => 'This is bar!' );
```


<br/>
<br/>

Cleary, not having a `new` operator, Raku refuses to do such a stupid thing with an error message like the following:

<br/>
<br/>
```shell
% raku test.p6 
===SORRY!=== Error while compiling /home/luca/tmp/test.p6
Unsupported use of C++ constructor syntax; in Raku please use method call syntax
at /home/luca/tmp/test.p6:7
------> my $foo = new Foo⏏( bar => 'This is bar!' );

```
<br/>
<br/>

What I have to do is to make my brain remember that the right way to do is `Foo.new`!
<br/>
But I find funny that Raku actually referes to C++ constructors as those used via the `new` operator.


## ?? !! Ternary Operator

I love the ternary operator, however it is quite hard for me to remember the `!!` part in Raku because I tend alsways to use the `:` common to many programming languages.


<br/>
<br/>
```perl6
#! raku

my $foo = "Foo";
say $foo ?? $foo : "NOT FOO";

```
<br/>
<br/>
and again, Raku being a patient instructor, reminds me about the error:

<br/>
<br/>
```shell
% raku test.p6
===SORRY!=== Error while compiling /home/luca/tmp/test.p6
Please use !! rather than :
at /home/luca/tmp/test.p6:5
------> say $foo ?? $foo :⏏ "NOT FOO";
    expecting any of:
        colon pair
```
<br/>
<br/>
