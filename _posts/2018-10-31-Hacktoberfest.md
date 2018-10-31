---
layout: post
title:  "Hacktoberfest 2018"
author: Luca Ferrari
tags:
- Hacktoberfest
- Open Source
permalink: /:year/:month/:day/:title.html
---
Another year, another october, another Hacktoberfest!

# Hacktoberfest 2018

## Glance at my Pull Request

Here is a short list about my Pull Requests and how they gone. The [overall status can be cheked as usual](https://hacktoberfestchecker.herokuapp.com/?username=fluca1978).

### `pgenv` multiple build support

In the last couple of weeks I spent some time working on `pgenv`, a PostgreSQL binary manager.
The idea behind `pgenv` is to build, select (use) and control a single instance of PostgreSQL on the local machine, allowing the user to keep more instances available at her fingertips.
Well, why not allow for building more than instance at the time?
This is what [this pull request is all about](https://github.com/theory/pgenv/pull/17): the `build** command accepts a list of PostgreSQL versions to build, so that it is possible to specify something like:

**I [closed](https://github.com/theory/pgenv/pull/17#issuecomment-425923970) this pull request because, after a discussion, we decided it was not so useful.**

```sh
% pgenv build 10.5 11beta4 9.6.5
```

and get a coffee.
The `remove` command has been changed similarly to allow the command to nuke more than one instance at the time.

### `pgTap` table inheritance support

In this [pull request](https://github.com/theory/pgtap/pull/177) I implemented a set of functions to test if a table has children. In particular, the following functions has been added to the `pgTap` suite:
- `has_inherited_tables` and its counterpart `hasnt_inherited_tables` that allows you to know if a table has been used as a parent table;
- `is_parent_of` and its counterpart `isnt_parent_of` to know if a table is n-th parent of another table;
- `is_child_of` and its counterpart `isnt_child_of` to know if a table is the n-th child of another table.

In particular the last two set of functions has been implemented using the recursive CTEs feature of PostgreSQL, that drop support to version 8.4 and not before.


### `pgTap` `col_is_pk()` variants

Anothe issue that caught my attention was [about adding some new overloaded variants of the function `col_is_pk()`](https://github.com/theory/pgtap/issues/133), that seemed a quite easy task. Therefore I created [a pull request to do exactly that job](https://github.com/theory/pgtap/pull/178), but while implementing it I found out that there was a potential clash between having arguments of type `name` and of type `text` (because essentially a `name` is a `char[63]`). This made failing tests in ambigous situations like:

```sql
SELECT col_is_pk( 'public', 'table', 'pk' );
```

because the server cannot choose between using a `col_is_pk( name, name, name )` and a `col_is_pk( text, text, tex )`.
By the way, I did a little refactoring and submitted the pull request, but oh gosh! I was testing on *decent* and *recent* versions of PostgreSQL, and the test suite was failing against 8.4 because of the introduction of `format()`. The point is I did not liked at all code like:

```sql
'Column ' ||quote_ident($1) || '.' ||quote_ident($2) || '(' || quote_ident($3)|| ') must be a primary key'
```

and thanks to `format()` (let's call it an SQL-like `printf(3)` for PostgreSQL, it could be rewritten in a very more readable way:

```sql
format( 'Column %I.%I(%I) must be a primary key', $1, $2, $3 );
```

The problem is that `format()` has been introduced in PostgreSQL 9.1, so all prior versions were failing. Argh! Shame on me, I swallowed the string concatenation and did another commit to fix it back.
But while doing all this stuff, I decided to take advantage of default arguments. Why having two almost identical representation of the same overloaded function? As an example:

```sql
CREATE OR REPLACE FUNCTION
col_is_pk( name, name, name, text )
-- implementation

CREATE OR REPLACE FUNCTION
col_is_pk( name, name, name )
-- implementation
```

The `text` last argument is the test description, that is not mandatory. But instead of having two functions, let's use a single one with a default to null optional argument. The implementation pattern looks like:

```sql
CREATE OR REPLACE FUNCTION
col_is_pk( name, name, name, text DEFAULT NULL )
-- implementation
   coalesce( $4, 'Column bla blah must be a primary key' )
```

So, if the optional `$4` text argument is provided, it is used, otherwise the `coalesce` function returns the default description.


### `www.itpug.org` trophey patches

Yeah, these are really simply patches I did because the maintainer was responsive that day, not because I'm still involved with the user group or have changed my mind about it. Essentially:
- [a little text over here and there](https://github.com/ITPUG/www.itpug.org/pull/5)
- [removal of the italian planet](https://github.com/ITPUG/www.itpug.org/pull/6), since it was not working anymore (and this commits to the fact ITPUG does not seem to be interested in keeping a planet, neither a blog update).


### `pgenv` patching feature

This [pull request](https://github.com/theory/pgenv/pull/20) was a proposal for allowing `pgenv` to automagically patch the source tree it is going to build. Before this patch, there was [some discussion](https://github.com/theory/pgenv/issues/18) about how to instrument `pgenv` for patching the source tree, and at that moment the script was doing a *direct patching* hardcoded into the script itself.

My proposal was to use an *index* file, that in turn contained a list of individual patches to apply on the source tree. But in order to let the user and the system to be as much flexible as possible, I decided to provide several optional indexes based on a PostgreSQL-version and Operating System combination, so that the user and the `pgenv` maintainers could choose how to distribute their patches and build a *patch-archive** different from any OS and/or PostgreSQL version or part of it.

**I [squashed this pull request](https://github.com/theory/pgenv/commit/865064af3782c5117303346a51a8c76bf06c5bb8) on October 25th**!

### `pgenv` message verbosity

This [pull request](https://github.com/theory/pgenv/pull/22) is a proposal for making `pgenv` configurable with regard to the amount of messages it does print. In particular the idea came into my mind from an [issue](https://github.com/theory/pgenv/issues/21) asking for some more configuration. The idea is that almost every message printed out by the program is now printed via a specific function, named `pgenv_message`, that prints to *stderr* and only if the `PGENV_VERBOSITY` variable has a level of verbosity greater or equal to that of the message.

This, of course, makes the usage of `pgenv_debug` obsolete, so that every call to `pgenv_debug` has been replaced to calls to `pgenv_message` with a level of `'debug'`.

### `perlbrew` clone-modules fix

This is related to some odd behavior of the command `clone-modules` that I implemented in the last year Hacktoberfest.
The problem was that such command was expecting two version numbers to perform the cloning, a source and a destination, but only the destination was effectively used, while the source was always erronously set to the currently running instance. This [pull request](https://github.com/gugod/App-perlbrew/pull/640) introduced the fix and also some extra check to avoid waste of time and resources while doing the cloning.


### `pgenv` build-git command

As requested in an [issue by Robert T.](https://github.com/theory/pgenv/issues/25) `pgenv` should gain the capability to build the current *development* version, that is the currently checkout out source tree.

Therefore I thinked about it for a couple of days and introduced the special version `dev` and the related command, so that `pgenv` was [able to fetch and build sources from the official git repository](https://github.com/theory/pgenv/pull/26). I pushed in the wild a little too fast, so I had to make some other commits later on.


## What I learned this year

This year was a little harder than the previous ones. Partially, it was because I committed to regularly propose my pull requests, that translates into a "less idea" principle. Second, because I tried to implement something more useful, dealing with new projects like `pgTap`. And it was from `pgTap` that I started learning how to deal with Continuos Integration and Travis.
