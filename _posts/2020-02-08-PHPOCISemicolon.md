---
layout: post
title:  "PHP & OCI: the bad semicolon"
author: Luca Ferrari
tags:
- php
- oracle
permalink: /:year/:month/:day/:title.html
---
I discovered with no little pain that OCI does not allow an ending semicolon.

# PHP & OCI: the bad semicolon

I don't know who to blame: PHP or the *Oracle Client Interface*?
<br/>
Surely the latter!
<br/>
I had some time wasted in trying to figure out why my query, issued by a PHP piece of code, were not hitting the Oracle database. Somehow, with a pure guess, I figured out that **ending semicolon is provoking OCI to fail** and `oci_error()` is of no help at all in finding this.
<br/>
<br/>
So I digged into the documentation, and [find out that SQL statements should not have an ending semicolon](https://www.php.net/manual/en/function.oci-parse.php){:target="_blank"}:

      SQL statements should not end with a semi-colon (";"). 
      PL/SQL statements should end with a semi-colon (";"). 
      
**What the hell is that?**      
Why isn't the driver (`oci`) cleaning the query string by itself? 
<br/>
**Why is semicolon used as a discriminator between *plain SQL* and *procedures*?**
<br/>
These are the times I thank PostgreSQL to be so much cleaner and consistent.
