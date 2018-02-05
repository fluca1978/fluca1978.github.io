---
layout: post
title:  "psql verbose quit"
author: Luca Ferrari
tags:
- postgresql
permalink: /:year/:month/:day/:title.html
---
A few days ago a new commit hit the `psql` code repository: there will be support for explicitly *exit*ing from the command prompt.

# `psql` and the *exit* command

If you are tied to the great `psql` command line tool, you know that in order to *exit* from such terminal you have to type a *pseudo-command*, in particular the `\q` command.

A few days ago, Bruce Momjian [commited a patch](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commitdiff;h=df9f599bc6f14307252ac75ea1dc997310da5ba6) that allows `psql` to understand also an explicit command typed as `exit ` or `quit`, as well as `help` (in substitution of `\h`).

This is a little but quite interesting enhancement, since a lot of terminals today, with particular regard to shells, understand an explicit quit command rather some sort of *internal command*.
It will not make my life easier, however, since I'm used to work with either `psql` or the `sqlite3` command prompt and I'm also miswriting the exit command using `\q` in the latter or `.quit` in the former. However, it will look a lot simpler for other users approaching `psql`!
