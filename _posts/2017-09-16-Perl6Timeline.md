---
layout: post
title:  A point of view about Perl 6 timeline
author: Luca Ferrari
tags:
- perl
- programming
permalink: /:year/:month/:day/:title.html
---
Why did Perl 6 took so many years to be released?

## A point of view about Perl 6 timeline
-----

It was July the 19 2000 (my birthday!) when the Perl 6 development was announced to the world, and it took until the Christmas eve in 2015
to produce the first official stable release. It rouglhy means 15 years of development.

It is a huge amount of time for a language version bump, except *this has not been a "simple" version number bump!*.
Yes, it is well known now that Perl 6 is not a new version of the Perl 5 language, but it is a new language itself. In other words, the term *Perl* does no refer to a single programming language anymore, rather to a *family of programming languages*, with two major players right now: **Perl5** and **Perl6**.
This is the short record of facts, period.

Now, let's come back to the original question: why did it took around fifteen years to get the new language see the light?
The reason is quite simple, maybe too simple, as stated by Larry Wall:

> In Perl 6, we decided it would be better to fix the language than fix the user

what that means is that Perl 6 did its best to produce a language that can hide the complexity of today's systems, leaving the developer
the pleasure to concentrate on concrete problems and not on boring details. If you think, this is something Perl has done for years: producing
a *flexible programming language that allows simple things to be simple and complex things to be possible*. Perl6 aims at being
a programming language that can make complex things not only possible, but somehow simple and most notably maintanable.

This required a great effort in design and re-design over and over the Perl core in order to achieve the state of the art as a programming language. And I'm not saying Perl6 is or will be the best language in the world, but it started with a simple aim and evolved to a very big one, and that lead to a complex development cycle in order to achieve it.

It is quite evident that Perl6 has been influenced by other popular languages, like Python or functional ones (while writing the compilers). Perl6 has done a very good job picking, all the fun from other languages. And I'm talking here about "cherry-picking", that would have been easy, rather Perl6 made other language features its own, adapted to the Perl culture and implemented them.

You can see it all over: from an easy embedded way to [access application arguments](https://docs.perl6.org/language/5to6-nutshell#Getopt::Long) or [data dumping widely available](https://docs.perl6.org/language/5to6-nutshell#Data::Dumper) to more powerful abstractions such as [roles](https://docs.perl6.org/language/objects#Roles), that are a kind of "interface on steroids".

*So, next time someone asks to you about the long timeline about Perl6, ask back how many time does it take to redesign everything so that
programming can be even more fun than how it is with Perl5!*
