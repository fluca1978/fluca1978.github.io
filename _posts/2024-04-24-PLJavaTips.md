---
layout: post
title:  "Using PL/Java: need for clarifications"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Sometimes it happens: I write something in a rush, and present it in a not-optimal way. And then I get advices!

# Using PL/Java: need for clarifications

On January, I wrote an article about [installing PL/Java on Rocky Linux](https://fluca1978.github.io/2024/01/17/pljavaPostgreSQL16.html){:target="_blank"}, and about some of the difficulties I had in achieving a fully operational installation, even if I did not dig enough into the problems that I encountered.

<br/>
**[Chapman Flack](https://github.com/jcflack){:target="_blank"}**, the most active developer in the project at the moment, take the time to write to me a very detailed email with a lot of suggestions for improvements and providing corrections to some of the misconceptions I present in such an article.

<br/>
I'm really glad to have received all those insights, and in order to spread the word, I'm writing here another article that, hopefully, fixes my mistakes.
I'm not following the same order that Chapman presented them to me, since in my opinion some issues are much more important than others, so I present from the most important to the least one, according to me.

## Editing the `java.policy` file

In my previous article, I advised readers to edit `java.policy` in the case there was a problem with Java permissions when executing PL/Java code.
Despite the fact that I clearly stated that relaxing the permissions to *all permissions* was not a good idea, Chapman emphasized two main problems in my example:
1) I was editing the **main** policy file, therefore changing the policy rules for **all** the Java code, not only for PL/Java one;
2) adding `java.security.AllPermission` made no distinction between trusted and untrusted languages.

Chapman pointed out that PL/Java uses a customized policy file, that can be found in the PostgreSQL configuration directory, hence in `$(pg_config --sysconfdir)`. This customizable configuration is available since PL/Java version 1.6, and is documented [here](https://tada.github.io/pljava/use/policy.html){:target="_blank"} in the section *"Permissions available in sandboxed/unsandboxed PL/Java"*.
This file defines two main principals:

<br/>
<br/>
```java
grant principal org.postgresql.pljava.PLPrincipal$Sandboxed * {
};


grant principal org.postgresql.pljava.PLPrincipal$Unsandboxed * {

        permission java.io.FilePermission
                "<<ALL FILES>>", "read,readlink,write,delete";
};


```
<br/>
<br/>

The first principal, the `PLPrincipal$Sandboxed` does not add any particular permission, while the `PLPrincipal$Unsandboxed` adds the permission to interact with the filesystem.

It is interesting to note that the `pljava.policy` file masks the `~/.java.policy` one (if exists), meaning that the latter is not used by PL/Java at all. However, the special property ` pljava.policy_urls` can be set to point and include additional (cumulative) policy files.

Conclusion: configuring the `pljava.policy` file is the right way to make permissions available to the PL/Java code in a fine grain manner, without having to deal with the system-wide set of permissions.


## Hopefully, there is no need to `SET pljava.libjvm_location`

Chapman provided me a link to the [PL/Java packaging documentation](https://tada.github.io/pljava/build/package.html){:target="_blank"} which contains a section named *"What is the default pljava.libjvm_location?"* that explains how package mantainers have information about where the default JVM installation is on the target system.
With such information, PL/Java pre-built packages could come pre-configured with the JVM location of the default installation on the system. So far, it seems the case for the Ubuntu package, while on my Rocky Linux it does not seem to be the case (or I messed the JVM installation).

Therefore, it is possible that there is no need to set `pljava.libjvm_location` if the package you installed already knows where the default JVM installation is on your operating system. However, knowing the aim of such variable and checking/configuring it allows database administrator to make PL/Java able to use a different (and specific) JVM.


## Using the `pljava-api` (locally)

In my previous post, I wrote that in order to compile Java code against PL/Java there is the need for the API jar installed, namely `pljava-api-x.y.z.jar`.
In order to get the API jar on the development machine, I wrote that you need to download the source code and compile it (using Apache `mvn`) and that this step is not simple at all, since it could require extra dependencies for the native code bindings.

Chapman pointed out that when you install the PL/Java from the PGDG distribution, you get also the above API jar installed on the PostgreSQL shared folder:

<br/><br/>
```shell
$ ls $(pg_config --sharedir)/pljava/*.jar
/usr/share/postgresql/16/pljava/pljava-1.6.7.jar
/usr/share/postgresql/16/pljava/pljava-api-1.6.7.jar
/usr/share/postgresql/16/pljava/pljava-examples-1.6.7.jar
```
<br/><br/>

Therefore, **there is no need to manually compile the API jar by yourself**, but you can use the one already installed into the PostgreSQL directory.

However, in order to make Apache Maven `mvn` aware of where the API jar is, you need to [install locally the JAR into the Maven repository](https://maven.apache.org/guides/mini/guide-3rd-party-jars-local.html){:target="_blank"}, so for example:

<br/>
<br/>
```shell
$ mvn install:install-file \
   -Dfile=$(pg_config --sharedir)/pljava/pljava-api-1.6.7.jar \
   -DgroupId=org.postgresql \
   -DartifactId=pljava-api \
   -Dversion=1.6.7 \
   -Dpackaging=jar
```
<br/>
<br/>

After the above, it is possible to compile Java code against the PL/Java API!



## Information in the `sqlj.jar_repository` table

The `sqlj.jar_repository` table contains the unique (short) name given to every installed JAR, as well as the location the JAR was loaded from (`jarorigin`):

<br/>
<br/>
```sql
testdb=# select jarname, jarorigin from sqlj.jar_repository;
 jarname |        jarorigin
---------+--------------------------
 PWC258  | file:///tmp/PWC258-1.jar
 PWC260  | file:///tmp/PWC260-1.jar
 PWC257  | file:///tmp/PWC257-1.jar
 PWC263  | file:///tmp/PWC263-1.jar
 pwc266  | file:///tmp/PWC266-1.jar
 PWC264  | file:///tmp/PWC264-1.jar
 PWC259  | file:///tmp/PWC259-1.jar
 PWC262  | file:///tmp/PWC262-1.jar
 PWC65   | file:///tmp/PWC265-1.jar
(9 rows)

```
<br/>
<br/>


In my previous article, I poorly explained this concept: when the `install_jar` function is executed it accepts as a afirst argument the URI from which the JAR is going to be loaded from, and such value is stored into the `jaroigin` field. **Once the JAR is deployed, such field does not have any useful meaning but giving information about the original location of the JAR**, and does not provide information about *where* the JAR currently is. For example, if on a local storage, the JAR file could even be removed, since `sqlj.install_jar` will *copy* the jar content into the database (I guess into `sqlj.jar_entry` table).


## Is there a round-trip of data between PostgreSQL and PL/Java?

Again, I poorly explained this concept in my previous article, stating that *"[...] using an *external* language like PL/Java means that PostgreSQL has to manage the round-trip of data between the database and the virtual machine, with the latter being fired at first execution."*

PL/Java exploits JNI to comunicate with the PostgreSQL backend process, and the comunication happens within the same process. Therefore there is no roundtrip, at least not as in involving a different process (i.e, inter-process comunication). However, there is still the need to properly convert complex data structures from Java types to PostgreSQL ones and viceversa, and that was what I meant with the wrong term "roundtrip".




# Conclusions

The above is set of details towards a better understanding of how PL/Java works. I have to admit that I'm really sorry about the probably worst mistake I did, that was to provide all the permissions to all the Java code running on the machine. It is embarassing, since I did also in the past a lot of work on the Java policy mechanism, but being so long since I don't develop Java anymore, I forgot all the good practice!

Besides, I hope this is going to better explain how to use PL/Java, and quite frankly I'm really happy to see that pretty much all my problems have a very strighforward solution that PL/Java developers have already addressed. This, again, emphasizes the maturity of such a project!
