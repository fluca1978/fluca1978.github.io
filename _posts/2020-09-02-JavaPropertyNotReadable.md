---
layout: post
title:  "JSF Property not readable on type boolean"
author: Luca Ferrari
tags:
- java
- programming
permalink: /:year/:month/:day/:title.html
---
I hate Java with a passion!

# JSF Property not readable on type boolean

refactoring an application, I decided to convert a `boolean` property mapped on a JSF UI radio button from a pure boolean to a three state choice: `null`, `true` and `false`.
<br/>
*How hard can it be?*
<br/>
<br/>
Depending on the Tomcat you are using, it can be *surprisingly hard*. In fact, passing a `null` to a *primitive boolean* value is wrong in Java, because autoboxing works only from objects to primitives, not viceversa.
<br/>
So the solution seems to be to convert your primitive type to an object one, that is a `Boolean`, but in doing so, **Java keeps things hard introducing again the awkward Java Beans specification**. Therefore, if you had `public boolean isFoo()` method, converting it to `public Boolean isFoo()` method will raise an exception that tells you the property `foo` is not readable.
<br/>
why?
<br/>
Because being `Foo` now an object, it must be accessed with `getFoo()` readable method.

# Summary

When dealing with Java objects that represent primitive boolean types, having the `is` method prefix is totally wrong.
<br/>
And it does not matter how ugly it is to have your Java code to be less readable, after all how ugly it is something like:

<br/>
```java
if( isDebugEnabled() )
   logger.debug( "Something" );
```

<br/>
<br/>
with respect to the horrible Java Beans one that follows?

<br/>
```java
if( getDebugEnabled() )
   logger.debug( "Something" );
```

<br/>
<br/>

One bovine solution could be to implement both `is` and `get` accessor with one relying on the other, but are you sure this will not produce confusion when dealing with code assistant during your daily development?


