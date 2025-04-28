---
layout: post
title:  "Oracle SQLPlus and sqitch: O/S Message: No such file or directory and SP2-0310"
author: Luca Ferrari
tags:
- sqitch
- oracle
permalink: /:year/:month/:day/:title.html
---
My (dis)adventures in Oracle-land using Sqitch.

# Oracle SQLPlus and sqitch: O/S Message: No such file or directory and SP2-0310

**I hate `sqlplus` with a passion!**

It is because I'm so used to the great PostgreSQL `psql` text client, that I don't see the point in all the *not-so-awesome* features (or should I call them *anti-features*?) that `sqlplus` provides.

Today, while trying to deploy a set of changes via `sqitch` (because I want to do things right, even when working with the "wrong" database!), I encountered a very strange error:

<br/>
<br/>
```shell
% sqitch deploy prod
Deploying changes to prod
  + my_change
Warning: View created with compilation errors.


Warning: View created with compilation errors.

O/S Message: No such file or directory
not ok
/opt/oracle/instantclient_21_11/sqlplus unexpectedly returned exit value 9
Deploy failed
```
<br/>
<br/>

Since I was unable to understand what that **`O/S Message`** meaning was, I opened an [issue on the sqitch discussion forum](https://github.com/sqitchers/sqitch/discussions/883){:target="_blank"}. Then, as I described in such issue, I tried to figure out what the error was.

In order to find out, I connected via `sqlplus` and then executed the file by means of:

<br/>
<br/>
```shell
% sqlplus ...
SQL> @deploy/my_change.sql
...
SP2-0310: unable to open file "param.sql"
SP2-0310: unable to open file "param.sql"
```
<br/>
<br/>


So, at least, I had an hint at what to search for: `param.sql` was the responsible.
Searching for such a string within my deploy file resulted in nothing, so I searched for `param`, finding out that there was a Doxygen style comment around a function:

<br/>
<br/>
```sql
/**
  * ...
  * @param id ...
  * @param error ...
  */
create or replace function ...
```
<br/>
<br/>

So I had *exactly two `@param` lines* and the errors were exactly two `param.sql` entries.

Therefore, `sqlplus` **was interpolating the comment `@param` as a command `@` for a file `param.sql`**. I changed the comments to read as `\param` and all worked fine.
