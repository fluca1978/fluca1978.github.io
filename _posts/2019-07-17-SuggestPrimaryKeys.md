---
layout: post
title:  "Suggesting Single-Column Primary Keys (almost) Automatically"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
Is it possible to infer primary keys automatically? If it, I'm not able at doing that, but at least I can try.

# Suggesting Single-Column Primary Keys (almost) Automatically

A comment on my previous [blog post about generating primary keys](https://fluca1978.github.io/2019/07/09/GeneratePrimaryKeys.html) with a procedure made me think about how to inspect a table to understand which columns can be candidates for primary keys.
<br/>
<br/>
Of course, this does make sense (at least to me) for *single-column* constraints only, because multi column constraint require a deep knowledge about the data. Anyway, [here it is my first attempt](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/suggest_primary_keys.sql):

```sql
CREATE OR REPLACE FUNCTION f_suggest_primary_keys( schemaz text DEFAULT 'public',
                                                   tablez text  DEFAULT NULL )
RETURNS SETOF text
AS $CODE$
DECLARE
  current_stats record;
  is_unique            boolean;
  is_primary_key       boolean;
  could_be_unique      boolean;
  could_be_primary_key boolean;
  current_constraint   char(1);
  current_alter_table  text;
BEGIN
  RAISE DEBUG 'Inspecting schema % (table %)', schemaz, tablez;

  FOR current_stats IN SELECT s.*, n.oid AS nspoid, c.oid AS reloid FROM pg_stats s
                       JOIN  pg_class c ON c.relname = s.tablename
                       JOIN  pg_namespace n ON n.oid = c.relnamespace
                       WHERE s.schemaname = schemaz
                       AND   c.relkind = 'r'
                       AND   n.nspname = s.schemaname
                       AND   ( ( s.tablename = tablez ))

  LOOP
    is_primary_key       := false;
    is_unique            := false;
    could_be_unique      := false;
    could_be_primary_key := false;
    RAISE DEBUG 'Inspecting table [%.%] (%.%) -> %', current_stats.schemaname,
                                                     current_stats.tablename,
                                                     current_stats.nspoid,
                                                     current_stats.reloid,
                                                     current_stats.attname;
     -- search if this attribute is already included into
     -- a primary key constraint
     SELECT cn.contype
     INTO   current_constraint
     FROM   pg_constraint cn
     JOIN   pg_attribute a ON a.attnum = ANY( cn.conkey )
     WHERE  cn.conrelid     = current_stats.reloid
     AND    cn.connamespace = current_stats.nspoid
     AND    a.attrelid      = current_stats.reloid
     AND    a.attname       = current_stats.attname;


     IF current_constraint = 'p' THEN
        is_primary_key := true;
     ELSE
       is_primary_key := false;
     END IF;

     IF current_constraint = 'u' THEN
        is_unique := true;
     ELSE
       is_unique := false;
     END IF;

     -- if this is already on a constraint, skip!
     IF is_primary_key OR is_unique THEN
        CONTINUE;
     END IF;

   -- check if this could be an unique attribute
   IF current_stats.n_distinct = -1 THEN
      could_be_unique := true;
   ELSE
      could_be_unique := false;
   END IF;

   -- could it be promoted as a primary key?
   IF could_be_unique AND current_stats.null_frac = 0 THEN
      could_be_primary_key := true;
   ELSE
     could_be_primary_key := false;
   END IF;

   IF could_be_primary_key THEN
      RAISE DEBUG 'Suggested PRIMARY KEY(%) on %.%', current_stats.attname,
                                                     current_stats.schemaname,
                                                     current_stats.tablename;
      current_alter_table := format( 'ALTER TABLE %I.%I ADD CONSTRAINT UNIQUE(%I)', current_stats.schemaname,
                                                                                    current_stats.tablename,
                                                                                    current_stats.attname );
   ELSE IF could_be_unique THEN
         RAISE DEBUG 'Suggested UNIQUE(%) on %.%', current_stats.attname,
                                                   current_stats.schemaname,
                                                   current_stats.tablename;
        current_alter_table := format( 'ALTER TABLE %I.%I ADD CONSTRAINT PRIMARY KEY(%I)',
                                                   current_stats.schemaname,
                                                   current_stats.tablename,
                                                   current_stats.attname );
    END IF;
  END IF;




   RETURN NEXT current_alter_table;
  END LOOP;

  RETURN;

END
$CODE$
LANGUAGE plpgsql;
```

The idea is to wrap into a function all the logic, so that I can pass either the schema or the table name to inspect and have the arguments be set to decent defaults.
<br/>
The first look is at `pg_stats` because it can provide hints about good candidates:
- if `n_distinct` is negative, and in particular is `-1`, the column has one different value on every different tuple, so it (as far as we know) *unique*;
- if `null_frac` is `0` the value is not null and it can be a candidate for a primary key.

<br/>
<br/>
**Of course, this means that the statistics must be up-to-date or the whole thing will not be able to suggest constraints!**
<br/>
<br/>

From `pg_stats` I get back column names, and the first thing to check then is if the column already appears in a constraint of type `p` (primary key) or `u` (unique); this prevents the function to suggest columns that already implied in such a  constraint, that is avoid suggesting obvious things.
<br>
<br/>
<br/>
The remaining is quite simple: if the column is already involved in a constraint, skip it; otherwise consider if it can be part of a `UNIQUE` or `PRIMARY KEY` constraint. Depending on the result, the right `ALTER TABLE` is emitted, so that the administrator can use it with rationality.
<br>
<br>
Here it is an example invocation:
```sql
testdb =# select * from f_suggest_primary_keys( 'respi', 'tipo_rensom' );
 DEBUG:  Inspecting schema respi (table tipo_rensom)
 DEBUG:  Inspecting table [respi.tipo_rensom] (151915.151952) -> pk
 DEBUG:  Inspecting table [respi.tipo_rensom] (151915.151952) -> id_tipo_rensom
 DEBUG:  Inspecting table [respi.tipo_rensom] (151915.151952) -> nome
 DEBUG:  Suggested PRIMARY KEY(nome) on respi.tipo_rensom
 DEBUG:  Inspecting table [respi.tipo_rensom] (151915.151952) -> descrizione
 DEBUG:  Suggested PRIMARY KEY(descrizione) on respi.tipo_rensom

                   f_suggest_primary_keys
 -------------------------------------------------------------------
 ALTER TABLE respi.tipo_rensom ADD CONSTRAINT UNIQUE(nome)
 ALTER TABLE respi.tipo_rensom ADD CONSTRAINT UNIQUE(descrizione)
```

With no surprise, the `pk` column, that is a `PRIMARY KEY` is inspected but skipped, while two other columns appear to be enough unique to take a role in a constraint addition.

<br/>
As you can imagine, this is just a little attempt in automating boring stuff. There is a lot of room for improvements, both on the performance way and on the more important support the function can provide to an administrator.
