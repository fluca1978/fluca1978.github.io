---
layout: post
title:  "Who needs comments?"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
Who cares about comments? Because you can certainly read your database schema, right?

# Who needs comments?

One [friend and colleague Enrico](http://www.pgtraining.com/chi-siamo/enrico-pirozzi){:target="_blank"} told me about one of those *hidden* features of `pg_dump`: **`--no-comments`**.
<br/>
<br/>
The option allows you to dump the database (or the part of it) without dumping any *user defined comment*, that is no comment on tables, data types, and nothing you placed with an explicit `COMMENT ON` statement.
<br/>
This made me lough at firts: why should I don't want comments on my dump? Are we still back in the ninenties where people thought that hiding information was a good strategy to ensure their job?
<br/>
However, there are some cases I can think about where you don't want comments. For example, some extensions use comments on objects to perform some *magic*, and `pgaudit` comes to mind. But it is not always true that you need to replicate the same configuration on another database, hence you should strip off the comments.

## How `pg_dump` avoids comments

Having a quick look at `pg_dump` source code, the function `dumpTableComment` represent a good introduction to how the comments are dumped or not. In particular, in the very beginning of the function, you can find something like:


<br/>
<br/>
```c
/* do nothing, if --no-comments is supplied */
if (dopt->no_comments)
	return;

/* Comments are SCHEMA not data */
if (dopt->dataOnly)
	return;

/* Search for comments associated with relation, using table */
ncomments = findComments(fout,
						 tbinfo->dobj.catId.tableoid,
						 tbinfo->dobj.catId.oid,
						 &comments);

```
<br/>
<br/>

If the `--no-comments` command line option is set (i.e., `dopt->no_comments` is true), the function returns immediatly since there is nothing to do. 
<br/>
Interestingly, if the user wants to dump only the data for the database, and not its schema, the comments are not dumped too. That's quite obvious if you think about.
<br/>
The `findComments` function is in charge of going to the storage to retrieve the comments, and it does in a *strange* way. It invokes, in turn, `collectComments`, that executes a query like the following one:

<br/>
<br/>
```c
appendPQExpBufferStr(query, "SELECT description, classoid, objoid, objsubid "
						 "FROM pg_catalog.pg_description "
						 "ORDER BY classoid, objoid, objsubid");
```
<br/>
<br/>

Do you see something strange there? *There is no `WHERE` clause in the query!* It does mean that the function is going to get all the comments from all the objects in the database, as reported also by the function comments preamble:

<br/>
<br/>
```c
/*
 * collectComments --
 *
 * Construct a table of all comments available for database objects.
 * We used to do per-object queries for the comments, but it's much faster
 * to pull them all over at once, and on most databases the memory cost
 * isn't high.
 *
 * The table is sorted by classoid/objid/objsubid for speed in lookup.
 */
```
<br/>
<br/>

The idea is that all the comments are retrieved on a single pass, and then `findComments` performs a kind of binary search to find out the exact range of comments that match the object it is dumping at that moment (i.e., the table).
