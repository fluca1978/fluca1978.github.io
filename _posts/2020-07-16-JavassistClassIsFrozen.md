---
layout: post
title:  "RuntimeException: class is frozen"
author: Luca Ferrari
tags:
- java
- javassist
permalink: /:year/:month/:day/:title.html
---
A run-time exception I cannot explain to myself.

# RuntimeException: class is frozen

I've used quite extensively [Javassist](https://www.javassist.org/){:target="_blank"} since when it was used for pretty much only *Aspect Oriented Programming*. Today, I would say that the strength of Javassist is to assist and implement magic under the hood of persistency layers, like [Hibernate](https://hibernate.org/){:target="_blank"}.
<br/>
<br/>
And it is from such a persisten layer, that a couple of times, luckily very rarely, I came across a `RuntimeException: class is frozen`.

## What is a frozen class?

[Javassist](https://www.javassist.org/){:target="_blank"} is a tool to modify at run-time Java classes. Once a class is loaded, Javassist *freezes* it, indicating the class should not be modified any more because it has been already loaded. In short, in order to modify a class at run-time you need to re-compile and use a different class loader.
<br/>
Javassist prevents you to do "stupid" things marking a class as sealed, but in order to manipulate it you need to *defrost* the class, that is always possible in order to proceed with changes at the bytecode level.

## Why is a frozen class throwing an exception?

I don't know, I can only guess that what is happening inside Hibernate is that the class need to be changed to insert *hooks** for Hibernate to handle persistency, but somehow Javassist prevents the system to modify the class.
<br/>
If I had to guess, I would say it is a class loader problem.

## How to fix the problem?

**Really, I don't know!**
<br/>
<br/>
However, restarting the application server (in my case an *Apache Tomcat*), fixed the problem. That lead me thinking it is a problem of caching of the classloaders.
