---
layout: post
title:  "Perl Weekly Challenge 107: copying myself"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 107: copying myself

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 107](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0107/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



## Eyes

Yesterday I did a check up of my eyes. 
<br/>
In short the situation is immutated, that is a good news at near 6 months from the last surgery. There is nothing to do right now, because it will be a risk for the long term, so the bad news is my sight is not going to imrove.
<br/>
Quite frankly it is hard for me to be at this sight-level, but I cannot change things, so I think I must get used to.



<a name="task1"></a>
## PWC 107 - Task 1

The first task was a repetition of the second task of [Perl Weekly Challenge 43](https://fluca1978.github.io/2020/01/20/PerlWeeklyChallenge_43.html){:target="_blank"}, one of the first challenge I did. 
<br/>
I'm not going to repeat the explanation right here, so please check what I wrote in the [in the second task of PWC 43](https://fluca1978.github.io/2020/01/20/PerlWeeklyChallenge_43.html){:target="_blank"}


<a name="task2"></a>
## PWC 107 - Task 2

The second task was about a reflection on a class: print out all available methods.
<br/>
This is really simple in Raku, thanks to the *Meta Object protocol* and the *Meta Object Class*. The only part, was to remember how to dynamically load an instance given a text name, but thanks to `mortiz` on IRC, I completed the exercise:


<br/>
<br/>
```raku
sub MAIN( Str $clazz = 'Rat' ) {
    .say for ::($clazz).^methods( :local ).sort;
}

```
<br/>
<br/>

The idea is simple: `$clazz` contains the name of the class we want to introspect, and thanks to the `::()` we can get its class. The `.^methods` activates the Meta Object and the `:local` adverb returns only a list of the methods defined within that class (i.e., not inherited). The other is just printing boilerplate.
