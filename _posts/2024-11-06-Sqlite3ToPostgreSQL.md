---
layout: post
title:  "PostgreSQL is super solid in enforcing (well established) constraints!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A note about mgirating from other databases...

# PostgreSQL is super solid in enforcing (well established) constraints!

Well, let's turn that around: *SQLite3 is somehow too flexible in allowing you to store data!*

We all know that.

And we all have been fighting situations where we have a well defined structure in SQLite3 and, ocne we try to migrate to PostgreSQL, a bad surprise arrives!
As an example, today I was trying to migrate a Django project with the built-in `loaddata` from a `dumpdata`, and sadly:


<br/>
<br/>
```
django.db.utils.DataError:
    Problem installing fixture '/home/luca/git/respi/respiato/sql/data.respi.json':
	   Could not load respi.PersonPhoto(pk=30647):
	       value too long for type character varying(20)
```
<br/>
<br/>


So in my SQLite3 tables some fields (at least one) have exceeded the size of the `varchar(20)`, and while PostgreSQL correctly refuses to store such value(s), SQLite3 happily get them into the database without warning you!

The fix, in this particular case, is quite simple: issueing an `ALTER TABLE personphoto ALTER COLUMN file_path SET VARCHAR(50)` does suffice. I could have used `text` also, but I would like to keep under control crazy values incoming from my application.

The point is: sooner or later, you will be stuck against a constraint your stack is not honoring, so be prepared for some troubles.

Using PostgreSQL in first place would have made the long-term maintanance easier, according to me.
