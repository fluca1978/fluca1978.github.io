---
layout: post
title:  "Template Databases"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PostgreSQL relies on the concept of template databases to create a new one.

# Template Databases

PostgreSQL relies on the concept of *template* as a way to create a new database.
The idea is similar to the one of the `/etc/skel` for Unix operating systems: whenever you create a new user, its own home directory is cloned from the `/etc/skel`. In PostgreSQL the idea is similar: whenever you create a new database, that is **cloned** from a template one.
<br/>
<br/>
PostgreSQL ships with two template database, namely `template1` and `template0`.
<br/>
`template1` is the default database, meaning that when you execute a `CREATE DATABASE` the system will clone such database as the new one. In other words:

<br/>
<br/>
```sql
CREATE DATABASE foo;
```
<br/>
<br/>
is the same as
<br/>
<br/>
```sql
CREATE DATABASE foo WITH TEMPLATE template1;
```
<br/>
<br/>

One advantage of this technique is that whatever object you put into the `tempalte1`, you will find into the new database(s). This could be handy when having to face multiple database with similar or identical objects, but can be a nightmare if you screw up your template database.
<br/>
Then there is `template0`, that is used as a *backup* for `template1` (in the case you screw up) or as a special templating database for handling particular situations like different encoding.

## Working with different templates

You can create your own template database, that you can then use as a *base* to create other database:

<br/>
<br/>
```sql
emplate1=# CREATE DATABASE my_template WITH
           IS_TEMPLATE = true;
CREATE DATABASE

template1=# CREATE DATABASE a_new_database
            WITH TEMPLATE my_template;
CREATE DATABASE
```
<br/>
<br/>

Having templates is handy, however is not mandatory to exploit a template to build a new database. Change the previous template so that it is no more a template database and then build another database:

<br/>
<br/>
```sql
template1=# ALTER DATABASE my_template
            WITH IS_TEMPLATE = false;
ALTER DATABASE

template1=# CREATE DATABASE a_new_database_from_no_template
            WITH TEMPLATE my_template;
CREATE DATABASE

```
<br/>>
<br/>

As you can see, you can use a normal (i.e., not template) database to build a new database too!
<br/>
This is possible only if done by a superuser!

<br/>
<br/>
```sql
template1=> CREATE DATABASE db_from_user;
CREATE DATABASE

template1=> CREATE DATABASE db_from_user_and_template
            WITH TEMPLATE my_template;
ERROR:  permission denied to copy database "my_template"
```   

<br/>
<br/>
As you can see, being a normal user you can create a new database using a template database, but not using a non-templating database.
<br/>
**Templates are exploitable by both normal and super users, but only super users can create a new database exploiting a database that is not marked as a template**.

## Connections while creating a database

When the `CREATE DATABASE` is performing, there must be no ther connections to the target database, it does not mean if it is a template or a normal database. The reason is that, in order to clone the database, there must be no activity on such database.


<br/>
<br/>
```sql
template1=> CREATE DATABASE db_from_user_while_template1_in_use;
ERROR:  source database "template1" is being accessed by other users
DETAIL:  There is 1 other session using the database.

```
<br/>
<br/>
Here it is: the message states clearly that there is some kind of activity on `template1` and therefore it is not safe to clone such database.
<br/>
The same happens with a non-template database:


<br/>
<br/>
```sql
template1=# CREATE DATABASE db_from_user_while_my_template_in_use
            WITH TEMPLATE my_template;
ERROR:  source database "my_template" is being accessed by other users
DETAIL:  There is 1 other session using the database.

```
<br/>
<br/>
It is interesting to note that it does not matter *what kind of activity* is ongoing in the database used as a template: it does suffice there is a single connection (event idle) to prevent `CREATE DATABASE` to continue.
<br/>
On the other hand, the system prevents any incoming connection to be established against the tempalte database until the `CREATE DATABASE` has finished and hence *releases* the database.


# Conclusions

Template database are used as a *skeleton* to be cloned when a new database is going to be created.
<br/>
The cluster can survive even without template database, but not having the default one(s) will make less comfortable the usage of `CREATE DATABASE`. You can build your own templates, and this is recommended to avoid tainting the default one(s), but you will need to specify your template name within every `CREATE DATABASE`.
<br/>
Last, the system will not allow you to use a database as a template if there are active connections (except your own), because cloning will become unsafe.
