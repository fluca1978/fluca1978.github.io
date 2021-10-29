---
layout: post
title:  "Java BigDecimal: are they equal?" 
author: Luca Ferrari
tags:
- java
- programming

permalink: /:year/:month/:day/:title.html
---
Comparing two `BigDecimal` in Java could not be as trivial as it seems.

# Java BigDecimal: are they equal?

How can you compar two numbers in Java?
<br/>
Well, it is quite simple, there is the `==` operator! That's true for simple types, not for objects, where the `.equals` method exists.
<br/>
Since `BigDecimal`s are objects, how do you test if two objects contain the same value? **Not using `.equals`!**
<br/>
That's true!
<br/>
Consider the following piece of code:

<br/>
<br/>
```java
BigDecimal a = new BigDecimal( "2.0" );
BigDecimal b = new BigDecimal( "2.00" );

if ( a.equals( b ) )
    System.out.println( "a equals b = " + a );

if ( a.compareTo( b ) == 0 )
    System.out.println( "a == b = " + a );
```
<br/>
<br/>

The first `if` branch evaluates to `false`, while the second evaluates to `true`.
<br/>
Why?
<br/>
The answer is in the [documentation for the `equals` method](https://docs.oracle.com/javase/10/docs/api/java/math/BigDecimal.html#equals(java.lang.Object)){:target="_blank"}:

<br/>
<br/>
```
Unlike compareTo, this method considers two BigDecimal objects equal only if they are equal in value and scale
```
<br/>
<br/>

It could have a sense in some *universe*, it does not make any sense to me because `2` is equal to `2.0`, that is equal to `2.00` and so one, without any regard of the scale.
<br/>
And just to take another look at the problem, how does Raku handle this? It gets the expected results:

<br/>
<br/>
```raku
% raku
To exit type 'exit' or '^D'
> 2 ~~ 2.0
True
> 2 ~~ 2.00
True
> 123456789.00000001 ~~ 123456789.000000010000000000
True

```
<br/>
<br/>

Therefore, the solution to compare two `BigDecimal`s in Java is to use the `compareTo` method, even if you *think the objects contain the same value*.

## Zero is not always Zero!

It can get even worst than the preceeding: how to compare *zero*?
Again, consider the following piece of code:

<br/>
<br/>
```java
BigDecimal a = new BigDecimal( "0.0" );
BigDecimal b = new BigDecimal( "0.00" );

// not working
if ( a.equals( b ) )
    System.out.println( "a equals b = " + a );

// working
if ( a.compareTo( b ) == 0 )
    System.out.println( "a == b = " + a );

// not working
if ( BigDecimal.ZERO.equals( a ) )
    System.out.println( "It is zero.zero!" );

// not working
if ( BigDecimal.ZERO.equals( b ) )
    System.out.println( "It is zero.zerozero!" );
```
<br/>
<br/>

And again, the `BigDecimal.ZERO` constant value cannot be easily compared to other values via the `equals` method!


# Conclusions

Seems to me Java is pretty much confused about number precision.
<br/>
While the test have been run against Java 11, I know this problem affects earlier (and probably later) versions. The fact is, from time to time, I forgot the semantic behind the usage of `BigDecimal` and stupidly try to *equal*-ize them.
<br/>
Oh poor me!
