---
layout: post
title:  "Inspecting Command Tags and Events in event triggers"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Event triggers are a very powerful mechanism to react to data structure changes in PostgreSQL.

# Inspecting Command Tags and Events in event triggers

While preparing an example for a course of mine about event triggers, I thought I've never proposed a *catch-all* event trigger debugging use case. So, here it is.
<br/>
Event Triggers are a powerful mechanism that PostgreSQL provides to react to database schema changes, like table or column addition and deletion, object creation, and so on. The [official documentatio](https://www.postgresql.org/docs/12/functions-event-triggers.html){:target="_blank"} already presents a couple of example about *dropping* objects or *rewriting* tables, so my little example is about more common commands. I create the following function:


```sql
CREATE OR REPLACE FUNCTION
f_event_trigger_demo()
RETURNS EVENT_TRIGGER
AS
$code$
DECLARE
event_tuple record;
BEGIN
   RAISE INFO 'Event trigger function called ';
   FOR event_tuple IN SELECT *
                      FROM pg_event_trigger_ddl_commands()  LOOP
                      RAISE INFO 'TAG [%] COMMAND [%]', event_tuple.command_tag, event_tuple.object_type;
   END LOOP;
END
$code$
LANGUAGE plpgsql;
```

It is quite simple to understand what it does: every time the function is triggered, it asks or the tuples out of the special function `pg_event_trigger_ddl_commands()`, that provides one tuple for every single command executed. Why multiple tuples? Because you could execute one command that explodes into different sub-commands.
<br/>
Than, simply, the function does print the command tag and the object type. 
<br/>
Usually *command tags* are *uppercase*, while *object types* are *lowercase*.
<br/>
The trigger can be created as follows:

```sql
testdb=# create event trigger tr_demo on ddl_command_end execute function f_event_trigger_demo();
```

It is now simple enough to test the trigger:


```sql
testdb=# create table foo();
INFO:  Event trigger function called 
INFO:  TAG [CREATE TABLE] COMMAND [table]
CREATE TABLE

testdb=# alter table foo add column i int default 0;
INFO:  Event trigger function called 
INFO:  TAG [ALTER TABLE] COMMAND [table]
ALTER TABLE

testdb=# create index idx_foo on foo(i);
INFO:  Event trigger function called 
INFO:  TAG [CREATE INDEX] COMMAND [index]
CREATE INDEX

testdb=# ALTER TABLE foo RENAME TO baz;
INFO:  Event trigger function called 
INFO:  TAG [ALTER TABLE] COMMAND [table]
ALTER TABLE

```

You can compare the output of the trigger function with the [event trigger firing matrix](https://www.postgresql.org/docs/12/event-trigger-matrix.html){:target="_blank"} to get an idea of what you can "catch".
<br/>
One last note: why have I attached the trigger to the `ddl_command_end`? Having a look at the [event trigger firing matrix](https://www.postgresql.org/docs/12/event-trigger-matrix.html){:target="_blank"} it looks like you can attach the trigger to either the `ddl_command_start` or `ddl_command_end` with the very same result, but the fact is that the function `pg_event_trigger_ddl_command()` works only on the *end* side of an event. The reason, as already explained, is that only approaching the *end* the system kows what a command has been exploded into.


## Source Code

You can find the source code of the trigger function [in my GitLab repository](https://gitlab.com/fluca1978/fluca1978-pg-utils/-/blob/master/examples/triggers/event_trigger_demo.sql){:target="_blank"}.
