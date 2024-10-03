---
layout: post
title:  "PostgreSQL adds the login type for event triggers"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Is it now possible to catch a login event.

# PostgreSQL adds the login type for event triggers

PostgreSQL 17 adds a new firing event for event triggers: [login](https://www.postgresql.org/docs/current/event-trigger-definition.html){:target="_blank"}. Therefore it is now possible to catch a login attempt on a database.

Caution: *this is not the same as Oracle logon triggers*, even if it resembles the same functionality to me.

However, thanks to this, is is now possible to get some more information when a login attempt succeeds.

In order to implement a poor-man auditing (**don't do this at home!**) to _breifly_ demonstrate this feature, you can:

<br/>
<br/>
```sql
postgres=# CREATE TABLE wrong_audit( pk int generated always as identity
          , who text
          , ts  timestamp default current_timestamp );
CREATE TABLE

postgres=# grant INSERT on table wrong_audit to public;
GRANT


postgres=# create or replace
	function f_etr_audit() returns event_trigger
	as $code$
	begin
     	insert into wrong_audit( who ) select current_role;
	end
	$code$
	language plpgsql;
CREATE FUNCTION

postgres=# create event trigger
	 poor_auditing on login
	 execute function f_etr_audit();
CREATE EVENT TRIGGER


```
<br/>
<br/>

Ano now, when you connect to the database you will see the table getting populated.


<br/>
<br/>
```sql
postgres=# table wrong_audit;
 pk |   who    |             ts
----+----------+----------------------------
  1 | postgres | 2024-10-03 11:39:27.659018
  2 | postgres | 2024-10-03 11:40:01.057011
  3 | postgres | 2024-10-03 11:46:06.38925
  4 | luca     | 2024-10-03 11:46:44.621835
  5 | luca     | 2024-10-03 11:46:46.389537
  6 | postgres | 2024-10-03 11:46:53.789339

```
<br/>
<br/>


There are a few things to note.

First of all, there is the need to grant the `INSERT` permission to the users that are going to fire the event, i.e., the user that are going to connect, or the trigger will not be able to execute. Obviously, there are other ways to do this, like settings permissions on the function itself.

Most important: if the trigger fails (due to an exception), the login attempt is aborted. For example, imagine that I remove the permissions on the tbale:

<br/>
<br/>
```sql
% psql -h localhost -U luca postgres
psql: error: connection to server at "localhost" (127.0.0.1), port 5432 failed: FATAL:  permission denied for table wrong_audit
CONTEXT:  SQL statement "insert into wrong_audit( who ) select current_role"
PL/pgSQL function f_etr_audit() line 2 at SQL statement

```
<br/>
<br/>

The connection is aborted due to the problem in completing the function.

Last but not least, the trigger function should not be a long running one, or the user will be locked waiting for the trigger to complete.


Now, for me to remember Oracle logon trigger, let's complicate a little the above example (**don't try this at home**):

<br/>
<br/>
```sql
postgres=# alter table wrong_audit add column db text;


ostgres=# create or replace function f_etr_audit()
returns event_trigger
as $code$
declare
        me text;
        db text;
begin
        SELECT current_role, current_database()
        INTO me, db;

        IF me = 'luca' AND db = 'postgres' THEN
           RAISE 'Get out of here!';
        END IF;

        insert into wrong_audit( who, db ) VALUES( me, db );
end
$code$
language plpgsql;

```
<br/>
<br/>


And now the poor bastard me when trying to connect to `postgres` gets:

<br/>
<br/>
```shell
% psql -h localhost -U luca postgres
psql: error: connection to server at "localhost" (127.0.0.1), port 5432 failed: FATAL:  Get out of here!
CONTEXT:  PL/pgSQL function f_etr_audit() line 10 at RAISE
```
<br/>
<br/>


while other users can still connect, and the table gets populated more and more.
