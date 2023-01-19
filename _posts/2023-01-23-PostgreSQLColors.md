---
layout: post
title:  "PostgreSQL command line colors!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A simple way to make more attractive the PostgreSQL command line interface!

# PostgreSQL command line colors!

Did you know that PostgreSQL tools can, under specific circumstances, display colors?
<br/>
Well, I didn't know until I came across [this section in the documentation](https://www.postgresql.org/docs/current/color.html){:target="_blank"} that explains it.
<br/>
There are **two different environment variables** named `PG_COLOR` and `PG_COLORS` respectively. The first (note the singular) decides if the colors have to be activated or not, while the second contains the sequence of colors.
<br/>
Clearly, colors are related to errors and other messages regarding *a tool* and not SQL errors!
<br/>

Let's see this in action:

<br/>
<br/>
<center>
<img src="/images/posts/postgresql/pg_colors.png" />
</center>
<br/>
<br/>

As you can see, after setting `PG_COLOR` to `always`, both `psql` and `pg_dump` show the error with a red color and the message tag with a bold face.
You can change the default color behaviour by setting the values in the `PG_COLORS` environment variable, so for example you can turn the errors to purple:

<br/>
<br/>
<center>
<img src="/images/posts/postgresql/pg_colors2.png" />
</center>
<br/>
<br/>

The `PG_COLORS` variable is a string that contains the log level (e.g., `error`) followed by the color code (e.g., `01;31` means bold red).
The same color palette that you apply in shell and `printf(2)` like escape sequences can be applied to `PG_COLORS` variable.
You can even make text blinking:

<br/>
<br/>
<center>
<img src="/images/posts/postgresql/pg_colors_blinking.gif" />
</center>
<br/>
<br/>

As far as I understand, the command line tools adapts to the colors thru the logging subsystem.
