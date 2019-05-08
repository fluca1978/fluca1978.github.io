---
layout: post
title:  "Why I hate ORMs: Hibernate and the duplicated rows on merge"
author: Luca Ferrari
tags:
- java
- hibernate

permalink: /:year/:month/:day/:title.html
---
While testing an application I was developing in Java, using Hibernate, I found some tuples on the database were duplicated instead of being updated.

# Why I hate ORMs: Hibernate and the duplicated rows on merge

Being myself a kind of old-school guy, I don't hype on ORMs (Object Relational Mappers), including the enterprise-class well know Hibernate.
<br/>
Today I was testing an application I was developing with Hibernate (5.4.1, if that matters), I found that instead of performing and `UPDATE` of a tuple, a new record (i.e., an `INSERT`** was performed. The result was having duplicated tuples on the database instead of a single one.
<br/>
**As horrible as it could sound, this is a great way to improve your database constraints!**
<br/>
Anyway, why the hell was Hibernate reasoning as it was a new tuple? Let's start from the piece of Java code responsible for the update (or supposed to be an update):

```java
dao.beginTransaction();
myOBJ = dao.merge( myOBJ );
dao.commit();
```

Easy enough, isn't it? **And that's why I hate ORMs with a passion: you have no way to understand what the ORM is supposed to do and what it will actually do in such a small snippet of code.**
<br/>
<br/>
A little more testing and I found out that the tuple was duplicated only *the first time*, and following `merge`s were done as `UPDATE`s in place. That is: the original tuple had some trouble in the beginning.
*Long story short*:**the `version` field on the table was `NULL`**.
<br/>
<br/>
Hibernate uses the `version` field on each table as an optimistic lock to understand if the tuple has changed in the meantime. And in this case, it was using such field to test if the tuple has been ever inserted, concluding that it had not been since the value was `NULL`. **Therefore the solution was to issue a global `UPDATE` to set the `version` field with a value of 0** and the application started to behave normally.
<br/>
<br/>

## Shame on me? Shame you Hibernate!

I let Hibernate/JPA to implement the tables in the database, so I did not write any DDL command at all. Why does this matter in the discussion? Because Hibernate was aware of the database (i.e., SQL "dialect") and was therefore able to issue a simple `DEFAULT 0` statement to attach to the column or, in a more fancy way, a trigger for an `INSERT` to populate such important field.
<br/>
<br/>
You see? That's why I hate ORMs like Hibernate: they force you to think as the ORM does, not to think at your data. Now, there are a few ORMs that are really better in my opinion because lower the impedence between the database and the object model, such as for example **DBIx**, but that's another story!
