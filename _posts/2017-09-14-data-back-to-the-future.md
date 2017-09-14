---
layout: post
title:  When your data is Back To The Future!
author: Luca Ferrari
categories: perl, programming
permalink: /:year/:month/:day/:title.html
---
Are your constraints preventing data moving back and forth time?

## When your data is Back To The Future!
-----
I had to write a simple Perl program to mangle some very old data. It was a quite easy task, after all, and it had to handle a few hundred ofmegabytes of pure text.
After the script ran, I took a look at the output data to see if everything was ok, but something hit my attention: *the very last tuples seemed to have been created after their modification date!*

Allow me to explain it better. Each tuple of text included a quite common set of information about creation and modification times, and a few of them had the latter (modification) in the past with respect to the former (creation).
Damn, how is it possible?

Let's exclude you have a [DeLorean with a *Flux Capacitor* on board](https://en.wikipedia.org/wiki/Back_to_the_Future).

The other chance was my Perl program was mangling data in the wrong way, mixing things. But again, let's exclude this since the program was a routinely script.

Another chance was that the *modify* and *create* fields had been flipped into the original data source, but that was not the case.

This lead to the only correct answer: __the data was wrong on its own!__

How was that possible? You know I had to understand what was going on, because knowing I'm doing the right stuff did not suffice, I had to prove someone else in the chain had tossed the data!
So I started to inspect how data was generated in the first place, and how it was updated.

What I discovered was a double mistake in how original data was handled:

1. first of all there was no constraint at all on the *modify* side of data, meaning that tiying such column to be greater or equal to
the *create* date would have been reduced the problem near to zero. Something as the following pseudo-SQL:


```sql
  create_date date default current_date,
  modify_date date default current_date,
  constraint in_the_future check( modify_date >= create_date )
````


2. data initialization. It happened that a program literally copied a tuple over another, adjusting the creation date but leaving the
modification one unchanged. This propagated the error thru the whole data set. Of course, application of the first constraint would have
prevented such propagation to happen, but even without such constraint a correct initialization of the modification date would have
prevented such problem.


Lesson re-learned: always initialize data the right way and place as much constraints as possible on place.
**It is better to have a program to fail due to a constraint than to have the whole data set corrupted!** While you can easily fix the failing program to respect constraints, you cannot adjust a corrupted data set.
  ````
