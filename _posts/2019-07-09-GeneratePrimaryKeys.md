---
layout: post
title:  "Generate Primary Keys (almost) Automatically"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
What if your database design is so poor that you need to refactor tables in order to add primary keys?

# Generate Primary Keys (almost) Automatically

While playing on quite large database (in terms of number of tables) with a friend of mine, we discovered that almost all tables **did not have a primary key**!
<br/>
Gosh!
<br/>
*This is really baaaad!*
<br/>
<br/>
Why is that bad? Well, you should not ask, but let's keep the poor database design alone and focus on some more concrete problems: in particular not having a primary key prevents a lot of *smart* softwares and middlewares to work on your database. As you probably know, almost every *ORM* requires each table to have at least one surrogate key in order to properly identify each row and enable persistence (that is, modification of rows).
<br/>
<br/>
Luckily, fixing tables for such software is quite simple: just add a surrogate key and everyone will be happy again. But unluckily, while adding a primary key is a matter of issuing an `ALTER TABLE`, doing so for a long list of tables is boring.
<br/>
<br/>
Here comes the power of PostgreSQL again: thanks to its rich catalog, it is possible to  automate the process.

<br/>
<br/>
In this post you will see how to build from a query to a whole procedure that does the trick.

## A query to generate the `ALTER TABLE` commands

A first example is the following query, that searches for every table in the schema `public` that does not have a constraint of type `p` (*primary key*) and issue an `ALTER TABLE` for such table:

```sql
testdb=# WITH
to_be_fixed AS
(
  SELECT c.relname,
  'ALTER TABLE '
  || quote_ident( n.nspname )
  || '.'
  || quote_ident( c.relname )
  || ' ADD COLUMN pk int GENERATED ALWAYS AS IDENTITY PRIMARY KEY;' AS command
  FROM pg_class c
  JOIN pg_namespace n ON n.oid = c.relnamespace
  WHERE n.nspname = 'public'
  AND   c.relkind = 'r'
  AND NOT EXISTS ( SELECT conname FROM pg_constraint WHERE contype = 'p' AND conrelid = c.oid )
  ORDER BY c.relname
)
SELECT command FROM to_be_fixed;
                                      command                                       
------------------------------------------------------------------------------------
 ALTER TABLE public.bar ADD COLUMN pk int GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
 ALTER TABLE public.foo ADD COLUMN pk int GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
```

So a first, desperate way of doing it is to adjust the above query to your schema, saving it to a file named `query.sql`, and then executing it putting the output into a text file (say `script.sql`) and then execute it. In other words, something like:

```shell
% psql -U luca -h miguel -f query.sql -o script.sql testdb
% psql -U -h miguel -f script.sql
```

But let's see a more tunable way of doing it.

## A function to generate the `ALTER TABLE` commands

I've written [a very small function to do the above `ALTER TABLE` commands](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/create_primary_keys.sql) in a way that is a little smarter and tunable.
The function accepts a couple of parameters, all with default values:
- `pk_prefix` (defaults to `pk`) the name of your primary key column, call it `id`, `pk` or whatever;
- `schemaz` (defaults to `public`) the schema where you want to operate on;
- `use_identity` true if you want to generate identity columns, false if you want to generate serial columns;
- `append_table_name` in order to avoid column name clashes (it could be you already have an `id` column somewhere), it is possible to append the table name to the column name `pk_prefix` so to generate almost unique keys.

The [function looks like the following](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/create_primary_keys.sql):

```sql
CREATE OR REPLACE FUNCTION f_generate_primary_keys( pk_prefix text DEFAULT 'pk',
                                                    schemaz text DEFAULT 'public',
                                                    use_identity boolean DEFAULT true,
                                                    append_table_name boolean DEFAULT false )
RETURNS SETOF text
AS $CODE$
DECLARE
  current_class pg_class%rowtype;
  current_alter_table text;
  current_pk_type text;
  current_pk_generation text;
  current_pk_name text;
BEGIN

  FOR current_class IN SELECT * FROM pg_class c
                       JOIN pg_namespace n ON n.oid = c.relnamespace
                       WHERE n.nspname = schemaz
                       AND   c.relkind = 'r'
                       AND NOT EXISTS ( SELECT conname FROM pg_constraint
                                        WHERE contype = 'p'
                                        AND conrelid = c.oid )

  LOOP
    RAISE DEBUG 'Table [%] without primary key', current_class.relname;

    current_pk_name := pk_prefix;
    IF append_table_name THEN
       current_pk_name := current_pk_name || '_' || current_class.relname;
    END IF;

    IF NOT use_identity THEN
       current_pk_type       := 'serial';
       current_pk_generation := '';
    ELSE
      current_pk_type       := 'int';
      current_pk_generation := 'GENERATED ALWAYS AS IDENTITY';
    END IF;



    current_alter_table := format( 'ALTER TABLE %I.%I ADD COLUMN %I %s NOT NULL %s PRIMARY KEY;',
                                   schemaz,
                                   current_class.relname,
                                   current_pk_name,
                                   current_pk_type,
                                   current_pk_generation );

   RAISE DEBUG ' -> %', current_alter_table;
   RETURN NEXT current_alter_table;
  END LOOP;

  RETURN;

END
$CODE$
LANGUAGE plpgsql;
```

Briefly, the function issues a query that is very similar to the above one, and that finds out all tuples in `pg_class` corresponding to a table without a primary key. For each table, the appropriate `ALTER TABLE` is built and issued as a returning value.
<br/>
Invoking the function produces the commands to execute after in the database:

```sql
testdb=# select * from f_generate_primary_keys( 'id', 'public', true, false );
                                   f_generate_primary_keys                                   
---------------------------------------------------------------------------------------------
 ALTER TABLE public.foo ADD COLUMN id int NOT NULL GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
 ALTER TABLE public.bar ADD COLUMN id int NOT NULL GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
(2 rows)

testdb=# select * from f_generate_primary_keys();
                                   f_generate_primary_keys                                   
---------------------------------------------------------------------------------------------
 ALTER TABLE public.foo ADD COLUMN pk int NOT NULL GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
 ALTER TABLE public.bar ADD COLUMN pk int NOT NULL GENERATED ALWAYS AS IDENTITY PRIMARY KEY;
(2 rows)

testdb=# select * from f_generate_primary_keys( 'id', 'public', false, true );
                        f_generate_primary_keys                         
------------------------------------------------------------------------
 ALTER TABLE public.foo ADD COLUMN id_foo serial NOT NULL  PRIMARY KEY;
 ALTER TABLE public.bar ADD COLUMN id_bar serial NOT NULL  PRIMARY KEY;
```


There is of course room for improvements, for instance executing the `ALTER TABLE` immediatly within the function.

## A procedure to execute the `ALTER TABLE` commands

It is now quite straightforward to wrap the `f_generate_primary_keys` function into a *procedure* and add transaction logic. The boring stuff is just to pass thru the arguments and control when to issue a commit while batch processing:

```sql
CREATE OR REPLACE PROCEDURE p_generate_primary_keys( pk_prefix text DEFAULT 'pk',
                                                     schemaz text DEFAULT 'public',
                                                     use_identity boolean DEFAULT true,
                                                     append_table_name boolean DEFAULT false,
                                                     commit_after int DEFAULT 10 )
AS $CODE$
DECLARE
  current_alter_table text;
  done int := 0;
BEGIN
  FOR current_alter_table IN SELECT f_generate_primary_keys( pk_prefix, schemaz, use_identity, append_table_name )
  LOOP
    RAISE DEBUG 'Executing [%]', current_alter_table;
    EXECUTE current_alter_table;
    done := done + 1;


    IF done % commit_after = 0 THEN
       RAISE DEBUG 'Forcing a commit';
       COMMIT;
    END IF;

  END LOOP;
  RAISE DEBUG 'Altered % tables in schema %', done, schemaz;
  COMMIT;
END
$CODE$
LANGUAGE plpgsql;
```

The important part here is, of course, the `EXECUTE` statement and the commit control.
Invoking the procedure proceduces something like:

```sql
testdb=# call p_generate_primary_keys( 'id', 'public', false, true );
DEBUG:  Table [foo] without primary key
DEBUG:   -> ALTER TABLE public.foo ADD COLUMN id_foo serial NOT NULL  PRIMARY KEY;
DEBUG:  Table [bar] without primary key
DEBUG:   -> ALTER TABLE public.bar ADD COLUMN id_bar serial NOT NULL  PRIMARY KEY;
DEBUG:  Executing [ALTER TABLE public.foo ADD COLUMN id_foo serial NOT NULL  PRIMARY KEY;]
DEBUG:  ALTER TABLE will create implicit sequence "foo_id_foo_seq" for serial column "foo.id_foo"
DEBUG:  ALTER TABLE / ADD PRIMARY KEY will create implicit index "foo_pkey" for table "foo"
DEBUG:  rewriting table "foo"
DEBUG:  building index "foo_pkey" on table "foo" serially
DEBUG:  Executing [ALTER TABLE public.bar ADD COLUMN id_bar serial NOT NULL  PRIMARY KEY;]
DEBUG:  ALTER TABLE will create implicit sequence "bar_id_bar_seq" for serial column "bar.id_bar"
DEBUG:  ALTER TABLE / ADD PRIMARY KEY will create implicit index "bar_pkey" for table "bar"
DEBUG:  rewriting table "bar"
DEBUG:  building index "bar_pkey" on table "bar" serially
DEBUG:  Altered 2 tables in schema public
LOG:  duration: 16.224 ms  statement: call p_generate_primary_keys( 'id', 'public', false, true );
CALL
```

Again, there is room for improvement, but this is just a quick demonstration of how easy it is to exploit PostgreSQL facilities to refactor your schema.
