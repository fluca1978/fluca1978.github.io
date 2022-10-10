---
layout: post
title:  "PostgreSQL ASCII numeric operators"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PostgreSQL has some special ways to provide numeric opeators by means of ASCII chars.

# PostgreSQL ASCII numeric operators

PostgreSQL has some *ASCII numeric representations* of commonly used numeric operators.
It could be not well know, since I suspect pretty much everyone is using the function operators, and moreover it is not so simple to [find them in the documentation by means of a searching for](https://www.postgresql.org/docs/14/functions-math.html){:target="_blank}.

<br/>

In any case, here they are:
- **`|/`** is the same as **`sqrt`**;
- **`||/`** is the same as **`cbrt`**;
- **`@`** is the same as **`abs`**.

<br/>
<br/>

An of course, it is quite easy to test such operators in action:

<br/>
<br/>

``` sql
testdb=> SELECT
            sqrt( 81 ) as sqrt,
            |/ 81 as root,
            cbrt( 1000 ) as cube_root,
            ||/ 1000 as root3,
            abs( -19 ) as abs,
            @ -19 as absolute;

-[ RECORD 1 ]-
sqrt      | 9
root      | 9
cube_root | 10
root3     | 10
abs       | 19
absolute  | 19

```
<br/>
<br/>

Quite frankly, I believe the function operators are more readable, in particular since I've never seen (yet) such operators in other programming languages.
