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
This is what [this pull request is all about](https://github.com/theory/pgenv/pull/17): the `build` command accepts a list of PostgreSQL versions to build, so that it is possible to specify something like:

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
