---
layout: post
title:  "Table name as function arguments: a few checks"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to check if a given table name exists and where to find it.

# Table name as function arguments: a few checks

Often I write some piece of code, usually a function or a procedure, that must operate dynamically on a table. To achieve this, I often pass the table name as an argument to the function.
<br/>
The function should always check that the table exists and, moreover, the function should always use the fully qualified name of the table to avoid schema conflicts and `search_path` pollution problems.
Last, sometime I use a relative name when I do pass the table as an argument, sometime I want to pass a fully qualified name to the function.
<br/>
<br/>

I've a template for doing this minimal checks, clearly it is just an idea on how to improve your own functions when dealing with table names.
Imagine a simple function that accepts a table name, as follows:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_do_stuff_on_table( t_name text )
RETURNS bool
AS $CODE$
DECLARE
	s_name text;
	info text[];
	pg_version  int;
	qualified_name text;
BEGIN


	-- parse the schema name
	info := parse_ident( t_name );
	IF array_length( info, 1 ) = 2 THEN
	   s_name := info[ 1 ];
	   t_name := info[ 2 ];
	ELSE
	   	-- try to understand if PostgreSQL 15 or higher
		SELECT setting::int
		INTO pg_version
		FROM pg_settings
		WHERE name = 'server_version_num';

		IF pg_version >= 150000 THEN
		   SELECT current_role
		   INTO   s_name;
		ELSE
		   s_name := 'public';
		END IF;

	END IF;


	-- check if the table exists
	PERFORM c.oid
	FROM pg_class c
	JOIN pg_namespace n
	ON n.oid = c.relnamespace
	WHERE c.relkind = 'r'
	AND   n.nspname = s_name
	AND   c.relname = t_name;

	IF NOT FOUND THEN
	   RAISE 'Table %.% does not exist, cannot proceed!', s_name, t_name;
	END IF;

	qualified_name := format( '%I.%I', s_name, t_name );
	RAISE DEBUG 'Table %', qualified_name;

	RETURN true;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The function accepts `t_name` that can be a relative name (e.g., *foo*) or an absolute name like *public.foo*.
<br/>
Initially the function exploits the `parse_identifier` internal PostgreSQL function to get out an array of elements, where the first one represents the schema name and the second one represents the table name. Thanks to this, and checking if the returned array has a size of 2, I can discriminate on what the function has received as an argument.
<br/>
If the function received a fully qualified table name, I store the schema into `s_name` and rewrite `t_name` with only its relative name, and nothing more has to be done on the naming part. On the other hand, if the function received a relative name, I must use a *default* schema, that generally speaking is `public` unless PostgreSQL 15, where it is the current role name. Therefore, I get the number of the PostgreSQL version and decide what value `s_name` will assume, either `public` or the `current_role` (interpolated) value.
<br/>
Once I have both the schema and the relative table name, I can check for the table in `pg_class`, assuming that `pg_namespace` confirms that the table is in such schema.
If the table is not there, I can `RAISE` an exception and stop the function right there, otherwise I can build a qualified name and go on with the rest of the work.
