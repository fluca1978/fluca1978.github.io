---
layout: post
title:  "Java Map: putIfAbsent"
author: Luca Ferrari
tags:
- emacs
- raku
permalink: /:year/:month/:day/:title.html
---
Is this method really worth?

# Java Map: putIfAbsent

I think it is pretty much impossible to write a good program without using, at least once, a `Map`, what other languages call *hash*.
<br/>
I tend to use a lot hashes in my applications, and often I find myself writing a pattern like the following one:

<br/>
<br/>
```java
if ( ! cache.containsKey( wanted ) )
   cahce.put( wanted, value );
   
value = cache.get( wanted );   
```
<br/>
<br/>

The idea is quite simple: if the value is already in map just get it, otherwise insert into the map and get it out. In this way, the map can work as a *cache* for a following elaboration.
<br/>
<br/>
Then a colleague of mine yelded about [Map.putIfAbsent()](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#putIfAbsent-K-V-URL ){:target="_blank"} method, that **apparently does** what I do, but in a single line of code.
<br/>
Emphasises on *apparently does*.
<br/>
There is a clear difference in my approach, which is not a *rocket-science* and the `putIfAbsent`: **the value to be stored could be obtained by a complex computation**, and therefore the following two blocks of code are not the same even if they look they are:

<br/>
<br/>
```java
cache.putIfAbsent( wanted, Database.doVeryLongQuery( wanted ) );
value = cache.get( wanted );   
```
<br/>
<br/>

<br/>
<br/>
```java
if ( ! cache.containsKey( wanted ) )
   cahce.put( wanted, Database.doVeryLongQuery( wanted ) );
   
value = cache.get( wanted );   
```
<br/>
<br/>

In fact, in the first snippet of code, the one that exploits the `putIfAbsent`, the complex database query is executed every time, while in the second example it is executed only if really required.
<br/>
There could be solutions that involves a lazy evaluation of the second argument to `putIfAbsent`, but they need to be manually crafted (i.e., the `Map` does not support them in its interface) and the ending result is to write a lot more code than the initial example.
