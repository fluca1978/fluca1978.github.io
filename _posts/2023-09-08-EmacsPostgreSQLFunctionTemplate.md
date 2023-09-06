---
layout: post
title:  "Using Emacs and YASnippet to quickly write PostgreSQL functions"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- emacs
permalink: /:year/:month/:day/:title.html
---
How a simple snippet can allow you to save time and improve your PostgreSQL code quality.

# Using Emacs and YASnippet to quickly write PostgreSQL functions

I love Emacs, and I also love PostgreSQL.
<br/>
Whenever I have to write PostgreSQL code, I use Emacs.
<br/>
Emacs can help me improving code quality, for example to write PostgreSQL functions. I use *YASnippet* as a package to provide the basic template for a PostgreSQL function.

# A PostgreSQL Function Template (in action)
Before explaining the concept, let's see a couple of short videos that demonstrate my snippet in action:


<br/>
<br/>
<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/9o5ahcmZK90?si=dRiQIsMn88ijuk5j" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<br/>

<iframe width="560" height="315" src="https://www.youtube.com/embed/y8YWMZGz5cY?si=ySvUEFgIOIYy0wP3" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</center>
<br/>
<br/>


# A PostgreSQL Function Template (the code)

The code for the template is the following one (I may change some bits here and there as time goes by):

<br/>
<br/>
```
# -*- mode: snippet -*-
# name: PostgreSQL Function
# key: function
# --
--
-- Function ${1:function_name}
-- Schema   ${2:public}
--
-- Description:
-- $3
--
-- Return Type: ${4:VOID}
--
CREATE ${5:OR REPLACE} FUNCTION
$2.$1($6)
RETURNS $4
AS $CODE$

$0

$CODE$
LANGUAGE ${8:plpgsql}
VOLATILE
;
```
<br/>
<br/>

The preamble is used from Emacs to understand the template name.
The following, is SQL code that works as a template for a function. Every `$n` placeholder is a tab stop that can be used to place the cursor within the text. For example, `${1:function_name}` is the first (`1`) tab stop, that present the default text `function_name` that is overwritten as I type in something. The name of function is then automatically replaced into the other `$1` placeholder.
<br/>
Note, how I first begin from the documentation, and then jump to the function code. This is a very important **added value: writing the documentation first I ensure every piece of code will have at least some documentation, and thanks to the placeholders, what I write in the documentation is used to name the function and its return type**.


# Conclusions

Emacs and YASnippet can be very powerful to help writing PostgreSQL code. While this post focuses on functions, it is possible to provide templates also for other kind of code schemes, like procedures, triggers, and so on.
