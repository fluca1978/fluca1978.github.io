---
layout: post
title:  "SQLPlus and multi-line input"
author: Luca Ferrari
tags:
- oracle
permalink: /:year/:month/:day/:title.html
---
As a client, SQLPlus is very poorly scriptable.

# SQLPlus and multi-line input

Today I was experimenting with the great [Sqitch](https://sqitch.org/){:target="_blank"} to manage changes to an Oracle database, and I was encountering a lot of problems.

It turned out that `sqitch` uses `sqlplus`, the default Oracle client, to execute commands on the Oracle database.
However, I was unable to execute my text scripts, since `sqitch` was always reporting errors. Therefore I started digging the problem, executing the script directly into `sqlplus`.

See an example:

<br/>
<br/>
```shell
% cat ~/tmp/table.sql
create table foo
(
  id number primary key

 , description varchar2( 10 )
);



% sqlplus -s luca/mysecretPassword@my_oracle:1521/database < ~/tmp/table.sql


...

SQL>   2    3    4  SQL> SP2-0734: unknown command beginning ", descript..." - rest of line ignored.
SQL> SP2-0042: unknown command ")" - rest of line ignored.

```
<br/>
<br/>

Note the totally unrelated error about `unknoqn command`. The problem is that `sqlplus` is unable to handle empty lines in a *sane* way.

Changing the original file removing the empty line makes it working:


<br/>
<br/>
```shell
% cat ~/tmp/table.sql
create table foo
(
  id number primary key
 , description varchar2( 10 )
);



% sqlplus -s luca/mysecretPassword@my_oracle:1521/database < ~/tmp/table.sql


...

SQL>   2    3    4    5
Table created.

```
<br/>
<br/>



After a little research, I found that there is a parameter named **`sqlblanklines`** that is turned off by default, and that denies the presence of blank lines in the SQL input. Setting it to `ON` allows the script to run fine.

There are two possible solutions:
- set the parameter as first thing in every script (poor practice)
- set the parameter in `~/login.sql`.

As an example:

<br/>
<br/>
```shell
 % cat ~/tmp/table.sql

set sqlblanklines ON;

create table foo
(
  id number primary key


 , description varchar( 10 )
);


% sqlplus -s luca/mysecretPassword@my_oracle:1521/database < ~/tmp/table.sql


...

SQL>   2    3    4    5
Table created.

```
<br/>
<br/>


The best approach is to create a `login.sql` script, that must be contained in a directory pointed by `$ORACLE_PATH` (do not confuse with `$ORACLE_HOME`!), for example my script is as follows:

<br/>
<br/>
```shell
% cat ~/bin/oracle/login.sql
--
-- Custom sqlplus customization script
--

prompt Running LUCA login script

set sqlblanklines ON;
set sqlprompt "_user'@'_connect_identifier >"

```
<br/>
<br/>



# Conclusions

As often, I find `sqlplus` a sub-optimal client for dealing with Oracle.
The fact, however, that a lot of tools expects it to be working to make their magics, and the fact that I don't fully understand how it works, can cause a lot of impedence!
