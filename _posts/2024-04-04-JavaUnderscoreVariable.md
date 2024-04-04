---
layout: post
title:  "Java 22 gets the underscore unnamed variable"
author: Luca Ferrari
tags:
- java
- perl
permalink: /:year/:month/:day/:title.html
---
The underscore variable arrives into the Java world!

# Java 22 gets the underscore unnamed variable

[JEP 456](https://openjdk.org/jeps/456){:target="_blank"} adds the **unnamed underscore variable** to Java.

It is that simple: you can use a **`_`** as a special variable name that can be used whenever you are not interested in the variable itself.

The simplest example, according to me, is the iteration loop. In a standard Java loop you would have something like the following:

<br/>
<br/>
```java
int counter = 0;
for ( Person p : people )
   counter++;
```
<br/>
<br/>

The above is clearly not the most efficient way to count how many people there are, but is a simple example: in the iteration the variable `p`, the iterator, is never used. The magic underscore variable allows you to avoid declaring an in-scope named variable, making the above as:

<br/>
<br/>
```java
int counter = 0;
for ( Person _ : people )
   counter++;

```
<br/>
<br/>

What is the advantage? **By usign an unnamed variable, you do not risk to mask out an already defined variable**.
<br/>
Period.
<br/>

I don't believe this really brings in readability, nor, as the [JEP 456](https://openjdk.org/jeps/456){:target="_blank"} provides as an example, allows for a shorter `try/catch` code style:

<br/>
<br/>
```java
try {
  ...
  }
  catch( IllegalArgumentException _ ) {
    return false;
  }
  catch( SQLException _ ) {
    return false;
  }

```
<br/>
<br/>

Exception must be handled in the correct way, and probably you need some information hidden in the exception itself, that you can access thru `_` but require handling.


So what is the point, after all?
<br/>

**Quite frankly, I don't see the reason for having `_`** in Java. And trust me, the **topic** of Perl (**`$_`**) does exactly what Java aims to do with `_`, so I'm luaghing at Java and all the other developers that criticized Perl for being *not readable due to its automagi variables*.


Having a *topic-like* unnamed variable in Java could be fun, but I'm seeing the utility because it does not reduce the typing needed.
Moreover, while in Perl the topic is clearly visible because of sigils, in Java (as well as in Python), the simple underscore can quickly get confused with some spellchecking symbol of the IDE.
