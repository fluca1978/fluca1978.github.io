---
layout: post
title:  "Oracle errors when executing multiple times a REVOKE (ORA-01927)"
author: Luca Ferrari
tags:
- oracle
permalink: /:year/:month/:day/:title.html
---
Another strange thing I saw when dealing with Oracle...

# Oracle errors when executing multiple times a REVOKE (ORA-01927)

This is another ehm...interering (?) thing I walked thru in my daily basis awkward Oracle experience: *if you execute multiple times a `REVOKE` on the same object, Oracle will complain about!*.
<br/>
Let's see an example:

<br/>
<br/>
```sql
REVOKE UPDATE,INSERT,DELETE ON foo FROM my_user;
-- so far, so good

REVOKE UPDATE,INSERT,DELETE ON foo FROM my_user;
ORA-01927: cannot REVOKE privileges you did not grant
```
<br/>
<br/>

Does it make sense?
<br/>
Well, yer and no.
<br/>
It surely does make sense to warn the user that it cannot revoke privileges anymore.
<br/>
On the other hand, it makes a lot harder to **automate** permission management, since this is a true error even if the situation has not changed at all!

<br/>
From my perspective, it is a lot more sane to emit a warning, not an error.
The opposite does not seem to happen with a `GRANT`, that applied multiple times to an object does not create an error.


<br/>
Just for the record: PostgreSQL behaves as I would expect, without throwing an error even in front of multiple executions of both `GRANT` and `REVOKE` statements!
