---
layout: post
title:  "org.hibernate.UnknownEntityTypeException: Unable to locate persister"
author: Luca Ferrari
tags:
- java
- hibernate
- eclipse
permalink: /:year/:month/:day/:title.html
---

I hate Hibernate with a passion, and this is just another stupid story where Hibernate caused me hours of madness.

# org.hibernate.UnknownEntityTypeException: Unable to locate persister

I hate Hibernate with a passion!
<br/>
If you have ever tried another ORM, like the great *DBI-Class*, you probably feel the same.
<br/>
<br/>
Anyway, while restoring a Java project from my hard disk, that faulted on the last week, I find this strange exception:

```
org.hibernate.UnknownEntityTypeException: Unable to locate persister 
```

followed by a class name. My application was using two different persistence sources, one was working just fine, the other was poducing the above exception, that as usual did not have any particular detail about the problem.
<br/>
Searching the web did not helped me in finding a solution.
<br/>
Finally, I made a little change in Eclipse: I set the *JPA Project* property and immediatly Eclipse did its job and highlighted a problem in my `persistence.xml` file.
<br/>
**My mapping had a *typo** in the package name of a few entities, and thus there was no correspondency between the class names and the mapped entities.**

<br/>
<br/>
Summary: the mapping was corrupted, maybe I restored a wrong checkout, and that prevented Hibernate to map the class. Of course, it could have helped a lot more to get information along the stack trace.
