---
layout: post
title:  "pg_dump and --if-exists little gem"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
An option I was not aware of...

# pg_dump and --if-exists little gem

`pg_dump` is a very useful tool to dump (and hence prepare to restore) a single PostgreSQL database.

When I use it, I usually add the options:
- `--clean` to `DROP` the database I'm dumping;
- `--create` to issue a `CREATE DATABASE` and reconnect to it.

Thanks to the above options, I'm pretty sure that I'm going to start over from a clean situation when restoring the dump.
This is particularly useful, according to me, when developing a new application and need to start over from scratch.


However, the result of the `--clean` option is that the SQL file begins, after the useual preamble, with something like:

<br/>
<br/>
```sql
DROP DATABASE miniondb;
```
<br/>
<br/>

While this is what I want, if I need to restore the backup on a fresh machine, where the target database was not already in place, the restore will cause a warning saying that the database cannot be dropped because it does not exist (yet).

And thic could be annoying from time to time!

But being PostgreSQL such a great advanced piece of software, `pg_dump` provides an option for add the very useful `IF EXISTS` to `DROP DATABASE`: **`-if-exists`** comes to the rescue!

<br/>
<br/>
```shell
% pg_dump --clean --create --if-exists ...
```
<br/>
<br/>

The above will result in the `DROP DATABASE miniondb IF EXISTS;`, that in turn will stop annoying me when the database is not already in place.

After all, the documentation for the `--clean` option already mentioned it clearly:

<br/>
<br/>
```shell
If any of the objects do not exist in the destination database,
ignorable error messages will be reported during restore,
unless --if-exists is also specified.
```
<br/>
<br/>

and much more on the option documentation itself:

<br/>
<br/>
```shell
--if-exists
     Use DROP ... IF EXISTS commands to drop objects in --clean mode. This suppresses “does
     not exist” errors that might otherwise be reported. This option is not valid unless
     --clean is also specified.
```
<br/>
<br/>


Note that `--if-exists` refers to *objects*, not only the whole database!
