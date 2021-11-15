---
layout: post
title:  "ORA-01461: can bind a LONG value only for insert into a LONG column"
author: Luca Ferrari
tags:
- oracle
- sql
permalink: /:year/:month/:day/:title.html
---
Another strange error by Oracle...

# ORA-01461: can bind a LONG value only for insert into a LONG column

Today, one of my Java applications starts having this strange error from Oracle:

<br/>
<br/>
```
ORA-01461: can bind a LONG value only for insert into a LONG column
```
<br/>
<br/>

that translates in italian as 

<br/>
<br/>
```
ORA-01461: si può eseguire associazione di valore LONG solo per inserirlo in una colonna LONG
```
<br/>
<br/>

The fact was that **I was not inserting any `long` value anywhere**, rather a `varchar2` value. I quickly got the idea that such text value was too long for the targeted column, and I was right: adjusting the size of the column and of the data inserted fixed the problem.
<br/>
However, it is really strange that Oracle (and/or the JDBC driver by Oracle) provides such an *idiot* error message!
