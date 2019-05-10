---
layout: post
title:  "The role of a role within another role"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A recursive title for a kind of recursive topic: what does really mean to have a role into another one? This article tries to figure out some basic knowledge about it.

# The role of a role within another role
After reading the very [excellent article by Hans-Jürgen Schönig](https://www.cybertec-postgresql.com/en/postgresql-using-create-user-with-caution/) about roles, I decided to provide my own vision about *users, groups and the more abstract role concept*.

## The word *role*
First of all, the word `role` has little to do with PostgreSQL: it is a word used in the SQL standard, so don't blame our favourite database for using the same word to express different concepts like *user* and *group*.

## Roles: are they users or groups?
The wrong part of the question is *or*: **roles are both users and groups**. Period.
A role is a stereotype, an abstraction for saying **a collection of permissions to do some stuff**. Now, often a collection of permission is granted to a user, and therefore a role smells like an user account, but in my opinion this is just a coincidence. And in fact, as in the best system administration tradition, when you have to assign a collection of permissions to more than one user you need a group; roles can therefore smell like a group.
<br/>
Remember: roles are collection of permission, what makes they smell as a group or an user is just **the way you use them**. If you use a role for a single user, then it is fine to think the role as an user account. If you use the role for more than one user, then it is fine to think the role as a group.
<br/>
Now, if you think this is trivial and simple, consider that a role can smell like an user and a group at the same time. **A role is a representative of a collection of permissions** and therefore can be something assigned to a single user, to a group (multiple users) or both. Somehow, it is like the chief of a company: he is playing at the same time as an employee and as an employer, as well as a representation of the company itself.

## Enough, let's see something!

Consider a very simple example: a school with a `schoolars` table that can be writen only by `professors` and read by other `students`: as you can image both `professors` and `students` will be groups of permissions.

```sql
testdb=# CREATE ROLE professors WITH LOGIN;
CREATE ROLE
testdb=# CREATE ROLE students WITH LOGIN;
CREATE ROLE

testdb=# REVOKE ALL ON schoolars FROM PUBLIC;
REVOKE
testdb=# GRANT ALL ON schoolars TO professors;
GRANT
testdb=# GRANT SELECT ON schoolars TO students;
GRANT
```


Anybody playing the `professors` role can do whatever he wants against the table:

```sql
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 professors
(1 row)

testdb=> TABLE schoolars;
 pk |     name     
----|--------------
  1 | Harry Potter
  2 | Luca Ferrari
(2 rows)

testdb=> INSERT INTO schoolars(name) VALUES('Ron Weasly');
INSERT 0 1
```

but anybody playing the `students` role cannot:

```sql
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 students
(1 row)

testdb=> TABLE schoolars;
 pk |     name     
----|--------------
  1 | Harry Potter
  2 | Luca Ferrari
  3 | Ron Weasly
(3 rows)

testdb=> INSERT INTO schoolars(name) VALUES('Rubeus Hagrid');
ERROR:  permission denied for table schoolars
```

So far, so good! But our groups are not very useful so far, they act as single accounts. Let's create a professor and add it to the `professors` group and see what happens:

```sql
testdb=# CREATE ROLE severus 
         WITH LOGIN 
         IN ROLE professors;
         
CREATE ROLE
```

The `IN ROLE professors` clause makes the role `severus` belonging to the `professors` group, and so we would expect it can do whatever the group can do:

```sql
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 severus
(1 row)

testdb=> TABLE schoolars;
 pk |     name     
----|--------------
  1 | Harry Potter
  2 | Luca Ferrari
  3 | Ron Weasly
(3 rows)

testdb=> INSERT INTO schoolars(name) VALUES('Drako Malfoy');
INSERT 0 1
```

So far so good, again!
However, the above example worked as expected because of the **default INHERIT behavior** as [clearly stated in the documentation](https://www.postgresql.org/docs/11/sql-createrole.html):

```
The INHERIT attribute is the default for reasons of backwards compatibility: 
in prior releases of PostgreSQL, users always had access to all privileges 
of groups they were members of. 
However, NOINHERIT provides a closer match to the 
semantics specified in the SQL standard.
```

### Role inheritance

When a role is *attached* to another role, and therefore is a member of the latter as if it was a group, PostgreSQL automatically uses the `INHERIT` property of the `CREATE ROLE`. Such property states that all permissions of the group the role is going to be a member will be forwarded to the member itself. In the above example, it does mean that `severus` has all the permissions of `professors` for free.
<br/>
But what happens if the role has been created without inheritance?

```sql
testdb=# CREATE ROLE severus WITH LOGIN IN ROLE professors NOINHERIT;
CREATE ROLE


testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 severus
(1 row)

testdb=> TABLE schoolars;
ERROR:  permission denied for table schoolars
```

**The role still owns all the permissions, but it explicitly needs to state which set of permission must eb applied** and this is done via a `SET ROLE` command:

```sql
testdb=> SET ROLE TO professors;
SET
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 professors
(1 row)

testdb=> TABLE schoolars;
 pk |     name     
----|--------------
  1 | Harry Potter
  2 | Luca Ferrari
  3 | Ron Weasly
  5 | Drako Malfoy
(4 rows)
```

It is like the role `severus` is allowed to become another user, like with system command `sudo(1)`, but explicitly needs to become such user.
In the case of `INHERIT` instead (the default behavior), all permissions are automatically granted.


### Dynamic behvaior

Let's add another professor, say `albus`, so that we will have `albus` that inherits from `professors` and `severus` who does not, but before that remove the `INSERT` permission from the `professors` group:

```sql
testdb=# REVOKE INSERT 
         ON schoolars 
         FROM professors;
REVOKE


testdb=# CREATE ROLE albus 
         WITH LOGIN 
         IN ROLE professors 
         INHERIT;
CREATE ROLE
```

Let's see what this mean at run-time:

```sql
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 severus
(1 row)

testdb=> INSERT INTO schoolars(name) 
         VALUES('Lord Voldemort');
ERROR:  permission denied for table schoolars
testdb=> SET ROLE professors;
SET
testdb=> INSERT INTO schoolars(name) 
         VALUES('Lord Voldemort');
ERROR:  permission denied for table schoolars


testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 albus
(1 row)

testdb=> INSERT INTO schoolars(name) 
         VALUES('Lord Voldemort');
ERROR:  permission denied for table schoolars
```

Neither `albus` nor `severus` can anymore insert a new tuple, as we would expect.
Now let's add again the `INSERT` permission to `professors`:

```sql
testdb=# GRANT INSERT 
         ON schoolars 
         TO professors;
GRANT
```

Let's see how both `severus` and `albus` can now perform an evil insert:

```sql
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 severus
(1 row)

testdb=> INSERT INTO schoolars(name) 
         VALUES('Lord Voldemort');
ERROR:  permission denied for table schoolars
testdb=> SET ROLE professors;
SET
testdb=> INSERT INTO schoolars(name) 
         VALUES('Lord Voldemort');
INSERT 0 1
```


```sql
testdb=> SELECT CURRENT_USER;
 current_user 
--------------
 albus
(1 row)

testdb=> INSERT INTO schoolars(name) 
         VALUES('Lord Voldemort');
INSERT 0 1
```

Did you spot the difference? **`INHERIT` means that the permission is immediatly granted at run-time to the role, while without inheritance the role must still become the target role to exploit the privileges**.

## Summary

So what is all about? When you create a role you can assign it to another role, that is make it belonging to a group. Such group must be enabled explicitly with a `SET ROLE` or, in the case of `INHERITANCE` all the permissions will be granted to the final user.
Remember: a role is just a collection of priviliges, and how you nest a role into another *merges* all the privileges, either flatting them (`INHERIT`) or keeping them separated (`NOINHERIT`).
