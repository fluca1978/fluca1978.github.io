---
layout: post
title:  "SQL-Plus and vertical spaces"
author: Luca Ferrari
tags:
- oracle
- sql
permalink: /:year/:month/:day/:title.html
---
SQL is a language where spaces are not-significant. Or are they?

# SQL-Plus and vertical spaces

Let's see a simple query I had to submit thru `sqlplus`:


```sql
UPDATE foo 
SET status = 200
WHERE id IN (

123,
456,
...
999 );
``**

Is the above query correct?
<br/>
**Yes, it is good because *SQL is space-agnostic* ** and therefore both horizontal and vertical spaces do not matter.
<br/>
<br/>
Unless you are using `sqlplus`, that considers the above a broken query because for some odd reason does not want an empty line. And in fact, removing the empty line before the first `IN` value fixes the problem.
