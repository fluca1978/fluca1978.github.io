---
layout: post
title:  "Oracle and adding a default value to a table column"
author: Luca Ferrari
tags:
- oracle
permalink: /:year/:month/:day/:title.html
---
Strange things happen when you are used to PostgreSQL and try to do the same in Oracle.

# Oracle and adding a default value to a table column

I placed a column in a table of mine, let's call it `foo`. Such column should have a numeric value, but I forgot to set up a `default` value for the column when I added to the table. The end result was that, querying the table, new tuples have a `null` value into such column.
<br/>
No matter, I thought, it is as easy as doing:

<br/>
<br/>
```sql
ALTER TABLE mytable MODIFY ( foo default 0 );

```
<br/>
<br/>

Ehm...no!
<br/>
In fact, while this was working for new tuples, it did not repair old `null` ones, so that I had to manually do a:

<br/>
<br/>
```sql
UPDATE mytable SET foo = 0 where foo is null;

```
<br/>
<br/>

Apparently, `default` definitions work differently in Oracle than in PostgreSQL, and most notably as I'm thinking they should work!
