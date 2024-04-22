---
layout: post
title:  "Installing PL/Java on PostgreSQL 16 and Rocky Linux"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pljava
permalink: /:year/:month/:day/:title.html
---
A short recap on some issues when dealing with PL/Java and Rocky Linux.

# Installing PL/Java on PostgreSQL 16 and Rocky Linux

It has been a while since I last used [PL/Java](https://github.com/tada/pljava){:target="_blank"}, and that's mostly due to the fact that I (luckily) use much more Perl (and hence, PL/Perl) in my everyday activity than Java.

I decided to implement a few functionalities exploiting Java, and so here it comes another installation of PL/Java. Installing on Rocky Linux has been a little tricky, so here it is a short recap about what to do.

I wrote about PL/Java in my book **[PostgreSQL 11 Server Side Programming Quick Start Guide](https://www.packtpub.com/big-data-and-business-intelligence/postgresql-11-server-side-programming-quick-start-guide){:target="_blank"}**.

As usual within the PostgreSQL ecosystem, PL/Java has [a very rich documentation](https://github.com/tada/pljava/wiki/User-guide){:target="_blank"}.


**IMPORTANT (2024-04-22): this article contains a few mistakes, that have been addressed in [my other article](https://fluca1978.github.io/2024/04/22/PLJavaInsightsAndFixes.html), so please ensure to read also the other article!**


## Is it worth?

I had to answer this question over and over: **is it worth using PL/Java for PostgreSQL triggers, functions, procedures and so on?**

As usual in these cases, there's no a single answer. First of all, using an *external* language like PL/Java means that PostgreSQL has to manage the round-trip of data between the database and the virtual machine, with the latter being fired at first execution. In short: performances are good but never as fast as native languages.
Second, PL/Java brings all the complexitly of a formally compiled language, therefore making changes to the code is not as simple as in other scripting languages.
Last, according to me, it does make sense if you need to bring some Java stuff into your scenario, either because it is a language you are absolutely proficient, or because you already have libraries and utilities that you don't want to convert in a database usable way.


## Installing PL/Java on Rocky Linux

Thanks to the PGDG, the official PostgreSQL repositories include an already available PL/Java package.
Therefore, installing PL/Java is as simple as:

<br/>
<br/>
```shell
% sudo dnf install pljava_16.x86_64
```
<br/>
<br/>

This makes the executable available, that is PostgreSQL will be able to run Java stuff within the database.

## Problem during compilation of PL/Java

If you need to develop against the PL/Java API, you need not only the executable, but also the whole library, that is compiled via *Apache Maven*. During the compilation, I got a few problems, most notably a `gssapi` related one.

I digged a little more using the `-X` flag:

<br/>
<br/>
```shell
% mvn -X clean install
...

In file included from /home/luca/pljava-1_6_6/pljava-so/src/main/c/InstallHelper.c:21:
/usr/pgsql-16/include/server/libpq/libpq-be.h:32:10: fatal error: gssapi/gssapi.h: No such file or directory
   32 | #include <gssapi/gssapi.h>
      |          ^~~~~~~~~~~~~~~~~
compilation terminated.

```
<br/>
<br/>

In order to solve the problem, I had to install the Kerberos development package:

<br/>
<br/>
```shell
% sudo dnf install krb5-devel.x86_64
```
<br/>
<br/>

and relaunching `mvn` worked as expected.



## Using PL/Java

In order to use PL/Java there could be the need to relax the JVM security constraints. I don't recommend to give an *all permissions*, but it is the quickest way to get PL/java able to run. Edit the file `/usr/lib/jvm/java/lib/security/default.policy` and make sure the very last section appears as follows:

<br/>
<br/>
```shell
// permissions needed by applications using java.desktop module
grant {
 permission java.security.AllPermission;
 ...
}
```
<br/>
<br/>

### Inform PostgreSQL and PL/Java about where the JVM is located

Before being able to use PL/Java there is the need to inform PostgreSQL about where the JVM is located (and hence, which).
This is achieved by a `SET` command:

<br/>
<br/>
```sql
testdb=# alter database testdb
     set
	 pljava.libjvm_location = '/usr/lib/jvm/java-11-openjdk-11.0.21.0.9-2.el9.x86_64/lib/server/libjvm.so';
```
<br/>
<br/>

and after this, it is possible to *install* PL/Java:

<br/>
<br/>
```sql
testdb=# create extension pljava;
```
<br/>
<br/>


### Install a JAR

PL/Java being Java, works on the concept of *jar* archives.
The JAR needs to be *installed* into PostgreSQL in order for PL/Java to be able to run its code. Installing a jar means that you need to inform PL/Java and PostgreSQL about the jar location.

<br/>
<br/>
```sql
testdb=> select sqlj.install_jar( 'file:///tmp/proj-0.0.1-SNAPSHOT.jar',
                                  'fluca',
								  true );
```
<br/>
<br/>

The first parameter to `install_jar` is the URI of the jar, the second is a shortname assigned to the jar and the last indicates if the deployment must be done.


### Set the classpath

Java has the notion of `classpath` and so does PL/Java. In order to use a function within an installed jar, there is the need to *map* the PostgreSQL schema to the Java classpath, in particular to the jar.

<br/>
<br/>
```sql
testdb=> select sqlj.set_classpath('public', 'fluca');
```
<br/>
<br/>

The jar named `fluca` will be added to the `public` PostgreSQL schema, so that when you refer to a method in the publica schema PL/Java will search within the `fluca` jar.

Assuming the jar contains the classic *Hello World* function, the final result is something like:

<br/>
<br/>
```sql
estdb=> \sf hello
CREATE OR REPLACE FUNCTION public.hello(towhom character varying)
 RETURNS character varying
 LANGUAGE java
AS $function$java.lang.String=com.example.proj.Hello.hello(java.lang.String)$function$

```
<br/>
<br/>

which makes very clear that `public.hello` is mapped to `Hello.hello` in the Java space.


# Where is my Java stuff?

PL/Java creates a schema `sqlj` that is used to handle both functions and tables that *route* stuff from PostgreSQL to Java and back.

In particular, `sqlj.jar_repository` contains an entry for every installed jar, so that you can for instance know where a jar is located:

<br/>
<br/>
```sql
estdb=> select jarid, jarname, jarorigin from sqlj.jar_repository;
 jarid | jarname |              jarorigin
-------+---------+-------------------------------------
     3 | fluca   | file:///tmp/proj-0.0.1-SNAPSHOT.jar

```
<br/>
<br/>

The table `sqlj.classpath_entry` shows how jar are *mapped* into PostgreSQL schemas:

<br/>
<br/>
```sql
testdb=> select r.jarname, r.jarorigin, c.schemaname
         from sqlj.jar_repository r join sqlj.classpath_entry c on c.jarid = r.jarid;
 jarname |              jarorigin              | schemaname
---------+-------------------------------------+------------
 fluca   | file:///tmp/proj-0.0.1-SNAPSHOT.jar | public

```
<br/>
<br/>

From the above it is possible to get the jar short name, the location of the jar on disk and to which PostgreSQL schema jar attributes have been mapped.

There are other interesting functions, like `get_classpath`, `set_classpath` and obviously `remove_jar` and `replace_jar`.


# Conclusions

PL/Java is a very powerful tool and an interesting language to extend the already rich set of features that PostgreSQL provides.
