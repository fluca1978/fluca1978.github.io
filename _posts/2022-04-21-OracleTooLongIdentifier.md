---
layout: post
title:  "ORA-00972 identifier is too long (well, not so long!)"
author: Luca Ferrari
tags:
- oracle
- java
permalink: /:year/:month/:day/:title.html
---
Oracle and its SQL Developer has strange behaviors...

# ORA-00972 identifier is too long (well, not so long)

Oracle prior to version `12` does not allow for an identifier name to be longer than *30 characters*.
<br/>
Period.
<br/>
There are ways to instrument Oracle to allow for longer names, but I'm not an Oracle DBA and I don't know the details.
However, sometimes I do forget this *max-length rule* and I do enter, in SQL Developers, identifiers such as column names that are sligthly longer in their names.
<br/>
So far so good, everything works, Hibernate is able to query columns with such long names, SQL Developer is able to do the same, and the long column name goes unnoticed.
<br/>
But one day, you want to run some SQL against such column, and the problem arise: **ORA-00972 identifier is too long**.
<br/>
There are several solutions to this problem:
- use a client able to deal with such column (e.g., SQL Developer);
- rename the column to be shorter;
- use SQLite3 (I'm joking!).


<br/>
What is really strange, to me, is that the same vendor products (Oracle) have a different behavior: SQL Developer allows for long column names but sql clients (e.g., `sqlplus`) do not.
