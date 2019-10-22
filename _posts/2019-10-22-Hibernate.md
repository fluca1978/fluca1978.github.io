---
layout: post
title:  "org.hibernate.AssertionFailure: null identifier**
author: Luca Ferrari
tags:
- java
- hibernate
permalink: /:year/:month/:day/:title.html
---
I spent a lot of time fighting against this exception!

# org.hibernate.AssertionFailure: null identifier

**I hate Hibernate with a passion**!
<br/>
More in general, I tend to hate *ORM* pretty much, because I feel like I need to adapt my applications and code to what the ORM thinks it should be. However, I have to use some ORMs, most notably Hibernate (I'm not reporting the version because it is awkwardly old).
<br/>
<br/>
Anyway, yesterday I start seeing this strange exception on one of applications of mine:

```java
ERROR     AssertionFailure                         <init> - an assertion failure occured (this may indicate a bug in Hibernate, but is more likely due to unsafe use of the session)
org.hibernate.AssertionFailure: null identifier
	at org.hibernate.engine.EntityKey.<init>(EntityKey.java:61)
	at org.hibernate.loader.Loader.extractKeysFromResultSet(Loader.java:688)
```

Searching the web did not provide me any hint about, and that is the reason why I'm writing this post.

## Trying to identify the problem

My first try was to enable as much logging as I could:

```java
log4j.logger.org.hibernate.SQL=debug 
log4j.logger.org.hibernate.type=trace 
```

but that does not revelead nothing interesting, since the problem wa not in a query rather in the query result. I then turned off the above variables and let Hibernate to log at a `DEBUG` level. That was enlightning because it revealed me a line like the following:

```java
DEBUG StatefulPersistenceContext   initializeNonLazyCollections - initializing non-lazy collections
```

Wait a minute: what does `initialize non-lazy collections` means?
<br/>
<br/>
I did a code scan on the entities just to find out I've got a collection **`@ManyToOne` with a fecthing type of `EAGER`**, and most notably the value I was trying to fecth was set to `null`.
<br/>
Yatze!
<br/>
So the workflow was as follows:
1) I did load entity `A`;
2) entity `A` has a `@OneToOne` relationship with entity `B`, so the latter is loaded by "reflection";
3) entity `B` has a `@ManyToOne` relationship with entity `C` and tries to `EAGER`-ly fetch it.

The point 3 is, of course, the problem.

## The solution

Having identified the problem, hitting the solution was quite easy: **change the `@ManyToOne`** fetching type from `EAGER` to `LAZY`. Please note that even setting `EAGER` with `optional = true` did not work.
<br/>
<br/>
Again, **I hate Hibernate with a passion**!
