---
layout: post
title:  "PL/pgSQL Trends"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A graph that shows the trends of `PL/pgSQL`, according to [Github.com](https://github.com){:target="_blank"}.

# PL/pgSQL Trends

I discovered [this excellent graphing system](https://tjpalmer.github.io/languish/#y=mean&names=sql%2Cplsql%2Cplpgsql){:target="_blank"} that shows several programming language trends, according to [Github.com](https://github.com){:target="_blank"}.
<br/>
<br/>
So, let's compare `PL/pgSQL` with `SQL` and `plSQL`:

<center>
<a href="https://tjpalmer.github.io/languish/#y=mean&names=sql%2Cplsql%2Cplpgsql"
   target="_blank">
   <img src="/images/posts/trends/2020_plpgsql.png" />
</a>
   
</center>


As you can see, the interest in `PL/pgSQL` has grown grown a lot in the last days, and this is due to the success PostgreSQL (and therefore the language) has achieved, at least in my opinion. I'm not sure that the comparison with `plSQL` is correct, because this is tied to a proprietary database and chances are there is less material available on Github, at least on a general basis.
<br/>
<br/>
The trends are confirmed also with regard to the issues, pull requests and stars, with the only exception that `plSQL` overtook `PL/pgSQL` on pull requests around the year 2014.
<br/>
<br/>
That's interesting, especially if you compare the trends with *real* programming language, I mean with multipurpose programming languages like `Perl`


<center>
<a href="https://tjpalmer.github.io/languish/#y=issues&names=sql%2Cplsql%2Cplpgsql%2Cperl"
   target="_blank">
   <img src="/images/posts/trends/2020_plpgsql_perl.png" />
</a>
   
</center>

<br/>
and everything disappear if you compare against *hype languages* like `Python` or...ehm...`Javascript`:


<center>
<a href="https://tjpalmer.github.io/languish/#y=issues&names=sql%2Cplsql%2Cplpgsql%2Cpython%2Cjavascript"
   target="_blank">
   <img src="/images/posts/trends/2020_plpgsql_python_javascript.png" />
</a>
   
</center>


