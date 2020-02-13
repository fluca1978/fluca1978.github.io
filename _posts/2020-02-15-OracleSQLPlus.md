---
layout: post
title:  "I'm not SQLPlus compatible!"
author: Luca Ferrari
tags:
- oracle
- sql
- postgresql
permalink: /:year/:month/:day/:title.html
---
My adventures with transactions, scripts and blank lines within SQLPlus. When even beginning transaction is not for the feebles!

# I'm not SQLPlus compatible!

This could be a flame, and to some extent it is. However, it must be said that after a lot of years using some tools, you get used to such tools and your mindset is somehow corrupted to think that's the normal (or conventional) way to behave.
<br/>
But that's not true: **that's the way your tools have used you**, and does not mean that such way is the only one, or the most correct one, or even a portable one.
<br/>
<br/>
Having said that, let's start the flame: **I hate Oracle SQL Plus**!
<br/>
After years of usage of PostgreSQL and `psql`, I'm probably used to its behaviour, that to me sounds very comfortable and reasonable, while on the other hand `sqlplus` seems really awkward.
<br/>
<br/>

## Blank Lines

I've spent a lot of time in figuring out why a well written and formatted SQL query was not working when I copy-and-paste into the `sqlplus` prompt.
<br/>
I then figured out that, somehow, *a blank line is something that is used to stop the current statement*.
<br/>
Dear `sqlplus**, have you ever spot that SQL is space-insensitive? And I mean, both *horizontal and **vertical** spaces*!
<br/>
But there's more: [there is a configuration option that should help](https://www.oreilly.com/library/view/oracle-sqlplus-the/0596007469/re89.html){:target="_blank"} in getting your statements parse-able even if they contain vertical spaces.
<br/>
Let's see this in action:

```sql
% $ORACLE_HOME/sqlplus 

SQL*Plus: Release 18.0.0.0.0 - Production on Thu Feb 13 23:13:50 2020
Version 18.3.0.0.0

Copyright (c) 1982, 2018, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production

SQL> SELECT CURRENT_TIMESTAMP
  2  
SQL> FROM DUAL;
SP2-0042: unknown command "FROM DUAL" - rest of line ignored.
SQL>
```

<br/>
As you can see, the error message `unknown command "FROM DUAL"` is **less than awesome** in helping me to understand what is wrong with my very simple query.
<br/>
But someone at Oracle was smart enough to introduce the `SQLBLANKLINES` setting:

```sql
SQL> SET SQLBLANKLINES ON
SQL> SELECT CURRENT_TIMESTAMP
  2  
  3  
  4  FROM
  5  
  6  DUAL
  7  
  8  ;

CURRENT_TIMESTAMP
---------------------------------------------------------------------------
13-FEB-20 11.16.23.805206 PM +01:00

```

Now, dear `sqlplus`, **why is this option not active by default?** Backward compatibity? Let the flag be available *when you really need such backward compatibility* for what matters.
<br/>
Moreover, why the option is so verbose? `SQL-BLANK-LINES`, why the `SQL`, do you expect non-SQL blank lines?
<br/>
<br/>

## Line Counter

There's another annoying thing: the *line counter*. Try to copy and paste the following query into you `sqlplus`:

```sql
SELECT
  CURRENT_DATE
  
  FROM
  DUAL
  ;
```

The result in the `sqlplus` terminal is as follows:

```sql
SQL> SELECT
  CURRENT_DATE
  
  FROM
  DUAL
  ;  2    3    4    5    6  

CURRENT_D
---------
13-FEB-20
```

Thos numbers `2    3    4    5    6  ` are the lines of the query as the terminal goes thru them. 
<br/>
Why is tha annoying? Because **copy and pasting an `sqlplus` session becomes a mess**. Think how often you copy and paste some statement and the output to report a problem, write an email, etc.
<br/>
<br/>

## Formatting columns

Last but not least, the worst one in my list is the way you have to specify the formatting of the columns.
Let's see a simple statement:

```sql
SQL> SELECT 0.1 + 0.2 AS SUM, 'Hello World' AS Greeting FROM DUAL;

       SUM GREETING
---------- --------------------------------
        .3 Hello World

```

Why all that space wasted for a single column? Why all that space wasted for the text column?
If you want to reduce the space waste, you need to issue `COLUMN FORMAT` commands specifying the name of the column. That's really stupid in my opinion and reminds me *Perl 5 formats*:

```sql
SQL> COLUMN GREETING FORMAT A5
SQL> SELECT 0.1 + 0.2 AS SUM, 'Hello World' AS GREETING FROM DUAL;

       SUM GREET
---------- -----
        .3 Hello
            Worl
           d
```

But the row wrapping makes the default output of a query a real mess. In the case you prefer something as the real Perl 5 formats, undefine the `WRAP` option:

```sql
SQL> SET WRAP OFF
SQL> SELECT 0.1 + 0.2 AS SUM, 'Hello World' AS GREETING FROM DUAL;

       SUM GREET
---------- -----
        .3 Hello

```

And what about the numbers? Again, a `COLUMN FORMAT` can help in getting some more decent results:

```sql
SQL> COLUMN SUM FORMAT '99.99'
SQL>  SELECT 0.1 + 0.2 AS SUM, 'Hello World' AS GREETING FROM DUAL;

   SUM GREET
------ -----
   .30 Hello

```


## Transactions

Transactions are what made me nuts.
<br/>
In PostgreSQL, and quite frankly also in SQLite and other *rational* (before *relational*) databases, a transaction is an unit of work that starts with a `BEGIN` and ends with either a `COMMIT` or  `ROLLBACK`.
<br/>
Of course there is the **auto-commit** mode, that makes a transaction implicit, but the point is transactions should be clearly defined by their boundaries.
<br/>
Period.
<br/>
**In `sqlplus` you are always within a transaction**, which is to me a very stupid thing.
That means you can issue a `COMMIT` or a `ROLLBACK` whenever you want, but it is not clear what you are committing or rolling back because you have to check manually the last part of your `COMMIT`/`ROLLBACK`.
<br/>
Let's see this in action:

```sql
SQL> insert into my_table( filename ) values ( 'foo' );

1 row created.

SQL> insert into my_table( filename ) values( 'bar' );

1 row created.

SQL> select filenam from my_table where filename in ( 'foo', 'bar' );
select filenam from my_table where filename in ( 'foo', 'bar' )
       *
ERROR at line 1:
ORA-00904: "FILENAM": invalid identifier


SQL> select filename from my_table where filename in ( 'foo', 'bar' );

FILENAME
--------------------------------------------------------------------------------
foo
bar

SQL> rollback;

Rollback complete.

SQL> select filename from my_table where filename in ( 'foo', 'bar' );

no rows selected
```


Some things to note:
1) the error query does not make the transaction to abort, that is unlike PostgreSQL you can keep your transaction working even if you introduce errors in the *transaction block*;
2) there is no `BEGIN`;
3) after a `ROLLBACK` both the `INSERT` are discarded, that makes it complicate because you have to reason when the `COMMIT`/`ROLLBACK` will apply.

<br/>
<br/>
But there are other tricks:
- a DDL command always performs a `COMMIT` before it is executed, so you could end up with consolidated data that you did not want to be;
- the previous is not really a problem because in Oracle DDL commands are not transactionals, therefore it does not make any sense to wrap them into a transaction;
- **a `BEGIN` block is used to start a PL/SQL code**.

<br/>
The last point is particularly important. I tend to produce programs that render some *portable* SQL code to perform bulk inserts, update, and alike. I instrument my programs to create transactions to control the workflow, and this means that my SQL scripts are produced with `BEGIN` and `COMMIT`.
<br/>
But when you feed such a script to `sqlplus** the prompt hangs. You could think it is working, but what is really happening is that it is waiting for other code to run.
<br/>
**This is stupid!**
<br/>
However, **stupidity seems to be in [SQL Standard](https://crate.io/docs/sql-99/en/latest/chapters/36.html){:target="_blank"}, not in Oracle by its own**: the standard defines that there is no need to open a transaction explicitly. And [PostgreSQL documentation also emphasizes this](https://www.postgresql.org/docs/12/sql-begin.html){:target="_blank"}:


```
BEGIN is a PostgreSQL language extension. 
It is equivalent to the SQL-standard command START TRANSACTION, 
whose reference page contains additional compatibility information.
```

That's another important point: the [SQL Standard](https://crate.io/docs/sql-99/en/latest/chapters/36.html){:target="_blank"} defines a `START TRANSACTION` optional way to mark the begin of a transaction. And here Oracle does things in nasty way, again:


```sql
SQL> START TRANSACTION;
SP2-0310: unable to open file "TRANSACTION.sql"
```

because `START`, shortcut `@`, reads a file and executes all the statements in it.

<br/>
<br/>
But Oracle awkardness does not stop to that: the [SQL Standard](https://crate.io/docs/sql-99/en/latest/chapters/36.html){:target="_blank"} dictates that in a transaction the time is discrete. In particular:

```
Second, the clock does not tick – CURRENT_TIME and all other 
niladic datetime function values 
are frozen throughout the life of a transaction.
```

<br/>
Frozen, uh? Now Oracle does not seem to implement `CURREN_TIME`, however does not seem to honor the frozen property too:

```sql
SQL> SELECT CURRENT_TIMESTAMP FROM DUAL;

CURRENT_TIMESTAMP
---------------------------------------------------------------------------
14-FEB-20 12.01.37.904216 PM +01:00

SQL> SELECT CURRENT_TIMESTAMP FROM DUAL;

CURRENT_TIMESTAMP
---------------------------------------------------------------------------
14-FEB-20 12.01.50.142446 PM +01:00
```

<br/>
<br/>
Last but not least, what happens when you exit the `sqlplus`? **On exit every transaction is committed!**
<br/>
There is a special parameter, named `exitcommit` that drives this option:

```sql
SQL> show exitcommit
exitcommit ON
```


### Transaction Oddities Recap

So, `autocommit` is usually `off`, meaning you have to explicit `COMMIT` your work.
<br/>
However, every DDL command will issue an *implicit `COMMIT`* for you (as per SQL Standard) and once you exit the `sqlplus` it will do another implicit `COMMIT` for you.
<br/>
Sounds not much coherent, uh?
<br/>
Time is not coherent, too, so I should say that is coherent in being not much coherent.




# Conclusions

It seems to me that `sqlplus` **requires a lot of effort to input and output queries** in a somehow *natural way* (i.e., how you would expect).
<br/>
While I'm not an `sqlplus` user, the above short experience does not make me wish doing more experiences using that.
<br/>
And transactions make me think I should stay away from Oracle databases!
