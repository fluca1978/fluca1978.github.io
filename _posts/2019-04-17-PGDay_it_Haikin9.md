---
layout: post
title:  "An article about pgenv"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv
permalink: /:year/:month/:day/:title.html
---
A few months ago I worked to improve the great `pgenv` tool by *theory*. Today, I try to spread the word in the hope this tool can grow a little more!

# An article about pgenv

### tl;dr
I proposed a talk about [`pgenv`](https://github.com/theory/pgenv), a Bash tool to manage several [PostgreSQL](http://www.postgresql.org) instances on the same local machine, to the Italian PGDay 2019. 
<br/>
*My talk has been rejected*, and I hate to waste what I have already prepared, so *I decided to transform my talk in an article, that has been quickly accepted on [Haikin9 Devops Issue](https://hakin9.org/product/practical-devops/)*!

<br/>
<br/>
I should have written about this a couple of months ago, but I did not had time to.
<br/>
My hope is that [`pgenv`](https://github.com/theory/pgenv) gets more and more users, so that it can grow and become someday a widely used tool. Quite frankly, I don't see this happening while being in Bash, for both portability and flexibility, and I suspect Perl is much more the language for a more flexible implementation. However, who knows? Gathering users is also a way to gather contributors and bring therefore new ideas to this small but very useful project.
<br/>
<br/>
In the meantime, if you have time and will, try testing the [build from git](https://github.com/theory/pgenv/pull/26) patch, that allows you to build and manage a development version of our beloved database.
