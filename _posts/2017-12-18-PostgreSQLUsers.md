---
layout: post
title:  "Creating many (really many) users in PostgreSQL"
author: Luca Ferrari
tags:
- postgresql
permalink: /:year/:month/:day/:title.html
---

PostgreSQL can really handle a lot of database users (*roles*), and it is possible to stress the system in a very simple way.

# Creating many (*really many*) users in PostgreSQL

In his post [Hans-Jürgen Schönig showed how to easily and quickly create a million users in PostgreSQL](https://www.cybertec-postgresql.com/en/creating-1-million-users-in-postgresql/); taking inspiration from such post, I decided to stress one of my virtual machines with a little more complex user creation use-case.

## Roles in Roles

One feature of PostgreSQL *roles* is that they can contain other roles, creating a hierarchy of roles. Therefore, I decided to write a simple `plpgsql` function to loop creating a chain of roles at each iteration. The function `f_users` accepts two integers:
- `deep` is the number of roles within the single role inheritance chain;
- `how_many` is the number of iterations.

As a result, the procedure will create `( 1 + deep ) x how_many` roles. Each role name is made by a random string and the iteration number, therefore preventing as much as possible collisions.

The function code is as follows:

```sql
CREATE OR REPLACE FUNCTION f_users( deep int, how_many int )
RETURNS VOID
AS
$BODY$
DECLARE
        main_role_name     text;
        current_role_name  text;
        current_level      int;
        iteration          int;
        query              text;
BEGIN

<<LP_MAIN>>
 FOR iteration IN 1..how_many LOOP
     -- main role
     main_role_name := 'role_test_' || md5( random()::text ) || '_' || iteration;
     RAISE DEBUG 'Main role is %', main_role_name;
     query := 'CREATE ROLE ' || main_role_name || ' WITH NOLOGIN CONNECTION LIMIT 0;';
     RAISE DEBUG '%', query;
     EXECUTE query;

     <<LP_DEEP>>
       FOR current_level IN 1..deep BY 1 LOOP
        current_role_name := main_role_name || '_' || current_level;
        RAISE DEBUG 'Level % -> role %', current_level, current_role_name;
        query := 'CREATE ROLE ' || current_role_name || ' WITH IN ROLE ' || main_role_name ||  ' NOLOGIN CONNECTION LIMIT 0;';
        RAISE DEBUG '%', query;
        EXECUTE query;
      END LOOP LP_DEEP;
 END LOOP LP_MAIN;

END;
$BODY$
LANGUAGE plpgsql;
```

Please note the above code can be optimized reducing the number of `RAISE` (that implies string concatenation).
The `connection limit 0` is for safety reasons: it is not desiderable to have such automatically created roles to be of any practical use.


## Results

The first attempt was short and sweet: 5000 roles within 1000 groups.

```sql
# SELECT f_users( 5, 1000 );
 f_users
---------

(1 row)

Time: 965.479 ms
```

As readers can see, this took less than a second to perform. What about a 10x factor?

```sql
# SELECT f_users( 5, 10000 );
 f_users
---------

(1 row)

Time: 9118.100 ms
```

It seems time is growing linearly.
Increase by a 5x factor:

```sql
# SELECT f_users( 5, 50000 );
 f_users
---------

(1 row)

Time: 104680.382 ms
```

To recap, the following is the timing of role creations:


| Groups   | Levels   |         ROLES |        TIME |
|:--------:| -------- |:-------------:|------------:|
| 1000     |    5     | 6000          | 1 sec       |
|          |    2     | 3000          | 0.3 sec     |
| 10000    |    5     | 60000         | 10 sec      |
|          |    2     | 30000         | 2.7 sec     |
| 50000    |    5     | 300000        | 105 sec     |
|          |    2     | 150000        | 36 sec      |


*for a total of __549000__ roles in __155 secs__.*



So time is not really increasing linearly, but as readers can see PostgreSQL can easily handle a half million roles in less than three minutes.
What about the virtual machine? Well, it is a *poor* `FreeBSD 11.1-RELEASE` running PostgreSQL 9.6 with 512 MB of RAM without WAL archiving or any other replication active. I cannot hit one million roles in a single shoot in such machine because it starts swapping until the swap daemon freezes.

In order to confirm such, let's consider how many roles there are in my system:

```sql
# SELECT count(*) FROM pg_roles;
 count
--------
 552018
```

the final result is greater than what is expected because I had already a discrete amount of roles.


*Not so bad for a database!*
