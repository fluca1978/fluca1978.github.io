---
layout: post
title:  "SQLite3 case insensitive LIKE: except on UTF-8!"
author: Luca Ferrari
tags:
- sqlite3
permalink: /:year/:month/:day/:title.html
---
Another "strange" behavior of SQLite3...

# SQLite3 case insensitive LIKE: except on UTF-8!

SQLite3 is a great tool, and I use often in my development environment for several reasons.
However, it has its own *quirks*, and I discovered another one, that is well documented: `LIKE` usually works in a case insensitive manner (pretty much as `ilike` in PostgreSQL).

Except when it has to deal with UTF-8 characters: in this case `LIKE` turns into a **case sensitive behavior** by default.

I found it by accident, while trying to fix a problem in an application I have to mantain, so I asked the [SQLite3 Community forum](https://sqlite.org/forum/forumpost/f254b4a254){:target="_blank"} for some explaination. As you can see from the example I posted, it seems that SQLite3 is not able to manage case-insensitive accented letters:

<br/>
<br/>
```sql
SQLite version 3.45.1 2024-01-30 16:01:20
Enter ".help" for usage hints.

sqlite> create table test( i varchar(10), c varchar(10) );
sqlite> insert into test( i, c ) values( 'perciò', 'PERCIÒ' );
sqlite> select * from test where i like '%perciò%';
perciò|PERCIÒ

-- not working
sqlite> select * from test where c like '%perciò%';

sqlite> PRAGMA case_sensitive_like=OFF;

-- not working
sqlite> select * from test where c like '%perciò%';

-- not working
sqlite> select * from test where i like '%PERCIÒ%';

-- working but without accented letters
sqlite> select * from test where i like '%PERC%';
perciò|PERCIÒ
```
<br/>
<br/>

The truth behind this behavior, is that the accented letters are managed as UTF-8, and in such cases the behavior of SQLite3 `LIKE` is to change transparently into **case sensitive**. This is [documented in section 5 of this page](https://sqlite.org/lang_expr.html){:target="_blank"} that states:

<br/>
<br/>
```
Note: SQLite only understands upper/lower case for ASCII characters by default. The LIKE operator is case sensitive by default for unicode characters that are beyond the ASCII range.
```
<br/>
<br/>


The solution is to either use an extension based on ICU, or to case fold the strings before comparing them.
