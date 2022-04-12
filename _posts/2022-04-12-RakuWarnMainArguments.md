---
layout: post
title:  "Raku and the MAIN arguments warn problem"
author: Luca Ferrari
tags:
- raku
permalink: /:year/:month/:day/:title.html
---
A problem of compatibility that I met when upgrading rakudo.

# Raku and the MAIN arguments warn problem

A few days ago I upgraded one of my machines, and this involved the upgrade of Rakudo from an *ancient* version (`2020-something`) to a recent release.
<br/>
So far so good, but my [Simple Jekyll Stat generator](https://github.com/fluca1978/jekyll-simple-stats){:target="_blank"} stopped working!
<br/>
*Argh!*
<br/>
<br/>
And that made me digging a little to find out what was wrong.

## The code

The code that generated the problem was like the following:

<br/>
<br/>

``` raku
sub MAIN( Str :$dir
            where { .so && .IO.d // warn "Specify the directory [$dir]" } ) {
    say $dir;
}

```
<br/>
<br/>

The problematic code is the `warn` call within the `where` condition in the `MAIN` argument checking.
The idea is to print a useful message when the argument `$dir` is set to a non directory, without having to deal with `USAGE` and friends.


## The problem

When running with Rakudo `2022-03`, I got something as follows:

<br/>
<br/>

``` raku
% raku ~/tmp/test.p6 --dir=foo
Use of uninitialized value of type Any in string context.
Methods .^name, .raku, .gist, or .say can be used to stringify it to
something meaningful.
 in block  at /home/luca/tmp/test.p6 line 4
Specify the directory []
 in block  at /home/luca/tmp/test.p6 line 4
Specify the directory [foo]
 in block  at /home/luca/tmp/test.p6 line 4
Use of uninitialized value of type Any in string context.
Methods .^name, .raku, .gist, or .say can be used to stringify it to
something meaningful.
 in block  at /home/luca/tmp/test.p6 line 4
Specify the directory []
 in block  at /home/luca/tmp/test.p6 line 4
Usage:
 /home/luca/tmp/test.p6 [--dir=<Str where { ... }>]
```
<br/>
<br/>

while when running with an ancient Rakudo (e.g., `2020-01`), it worked as expected.
<br/>
One interesting thing to note in the above output, is that the **`warn` is called three times (or in general, multiple times), with only the second one with a valued `$foo`**.
<br/>
The fact that `where` can be called multiple times is [specified in the documentation](https://docs.raku.org/type/Signature#index-entry-where_clause){:target="_blank"}:

<br/>
<br/>

``` raku
The code in where clauses has some limitations: anything that produces side-effects (e.g., printing output, pulling from an iterator, or increasing a state variable) is not supported and may produce surprising results if used. Also, the code of the where clause may run more than once for a single typecheck in some implementations.
```
<br/>
<br/>


## The solution

Interestingly, the solution to the problem, that allows to run the same code with a recent Rakudo, is to **explicitly indicate the variable to check in the `where` clause, therefore not using the topic**:


<br/>
<br/>

``` raku
sub MAIN( Str :$dir
            where { $dir.so && $dir.IO.d // warn "Specify the directory [$dir]" } ) {
    say $dir;
}

```
<br/>
<br/>


# Conclusions

**You shold always stay up-to-date**, so that your application is battle-tested against current version of Raku and Rakudo.
<br/>
Besides this trivial fact, sometimes Raku makes things difficult to understand, or better, Raku is great to understand what you want to do even when you *don't write it*, but sometimes you need to be clearer!
