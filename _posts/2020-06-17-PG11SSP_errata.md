---
layout: post
title:  "PostgreSQL 11 Server Side Programming Errata Corrige"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A reader provided us a feedback about a wrong listing.

PostgreSQL 11 Server Side Programming Errata Corrige
---

I have already written about how my first book on PostgreSQL, named **[PostgreSQL 11 Server Side Programming Quick Start Guide](https://www.packtpub.com/big-data-and-business-intelligence/postgresql-11-server-side-programming-quick-start-guide)**, gained more attention.

<br/><br/><br/>

<center>
<a href="https://www.packtpub.com/big-data-and-business-intelligence/postgresql-11-server-side-programming-quick-start-guide" target="_blank" >
<img src="/images/posts/pg11ssp/cover.png" />
</a>
</center>

<br/><br/><br/>
Gaining attention also means that readers could find out problems and errors, **and this is good** (to me)!
<br/>

The first problem that has been reported to me is described here, so that if you are reading the book can better understand and deal with the problem.

<br/><br/>
## Listing 8 on Chapter 3

The *Listing 8* in chapter 3 is wrong, and in particular it is the very same listing as *Listing 13* later in the chapter.
The problem is that the shown listing 8 does not include a variable, namely `file_type`, that is referenced in the text.
<br/>
Therefore, if you are dealing with that particular example, please consider that the right listing is reported [on the official GitHub repository](https://github.com/PacktPublishing/PostgreSQL-11-Quick-Start-Guide/blob/master/Chapter03/Chapter03_Listing08.sql){:target="_blank"}.


<br/><br/>
I'm really sorry about the misplaced listing, I hope this can help making it more readable.
