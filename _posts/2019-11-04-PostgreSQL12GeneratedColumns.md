---
layout: post
title:  "PostgreSQL 12 Generated Columns"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
PostgreSQL 12 provides support for automatically computed columns.

# PostgreSQL 12 Generated Columns

PostgreSQL 12 introduces the [generated columns](https://www.postgresql.org/docs/12/ddl-generated-columns.html), columns that are automatically computed depending on a *generation expression*. 
<br/>
The usage of generated columns is quite simple and can be summarized as follows:
- the column must be annotated with the `GENERATED ALWAYS AS (...) STORED` instruction;
- the expression in parentheses must use only `IMMUTABLE` functions and cannot use subqueries.
<br/>
For more specific constraints, see the [official documentation](https://www.postgresql.org/docs/12/ddl-generated-columns.html).
<br/>
<br/>


Please note I've indicated the `STORED` clause because at the moment PostgreSQL supports only that kind of columns: a `STORED` generated column is saved on disk storage as a normal column would be, the only difference is that you cannot modify it autonomously, the database will compute it for you.
<br/>
<br/>
You can think of a stored generated column as a trade-off between a table with a trigger and a materialized view. When the `VIRTUAL` (as opposed to `STORED`) will be implemented, the column will take no space at all and will be computed on each column access, something similar as a view.

## An example of not-generated column

Let's see generated columns in action: consider an ordinary table with a dependency between the `age` column and the `birthday` one, since the former can be computed from the values in the latter column:

```sql
testdb=> CREATE TABLE people( 
                  name text, 
                  birthday date, 
                  age int );

testdb=> WITH year AS ( 
   SELECT ( random() * 100 )::int % 70 AS y 
)
INSERT INTO people( name, age, birthday )
SELECT 'Person ' || v, y, current_date - ( y * 365 )
FROM generate_series(1, 1000000 ) v, year;
```

Let's see how much space does it occupy to have such table filled with one million of rows:

```sql
testdb=> SELECT pg_relation_size( 'people' );
 pg_relation_size 
------------------
         52183040
```


## An example with generated columns

In order to create a similar table where `age` is automatically computed. 
<br/>
Since the column must use an `IMMUTABLE` function, the first step is to abstract the computation into a function:

```sql
testdb=> CREATE OR REPLACE FUNCTION 
f_person_age( birthday date )
RETURNS int
AS $CODE$
BEGIN
    RETURN extract( year FROM CURRENT_DATE )
           - extract( year FROM birthday )
           + 1;
END
$CODE$
LANGUAGE plpgsql IMMUTABLE;
```

Then it is possible to create the table using the function as *generation method*:

```sql
testdb=> CREATE TABLE people_gc_stored ( 
      name text, 
      birthday date, 
      age int GENERATED ALWAYS AS ( f_person_age( birthday ) ) STORED
  );
```

If the table is filled in a similar way, the space occupied is the same:

```sql
testdb=> INSERT INTO people_gc_stored( name, birthday )
         SELECT 'Person ' || v, current_date - v 
         FROM generate_series(1, 1000000 ) v;

testdb=> SELECT pg_relation_size( 'people_gc_stored' );
 pg_relation_size 
------------------
         52183040

```

Why using a function in the generated column? Because if we place the real expression we got an error at creation time:

```sql
testdb=> CREATE TABLE people_gc_stored ( 
      name text, 
      birthday date, 
      age int GENERATED ALWAYS AS ( 
              extract( year FROM CURRENT_DATE ) 
              - extract( year FROM birthday ) 
              + 1 ) STORED
  );
  
ERROR:  generation expression is not immutable
```


## Writing the generated column

As already written, the generated column is not writable once it has been computed:


```sql
testdb=> UPDATE people_gc_stored SET age = 40;
ERROR:  column "age" can only be updated to DEFAULT
DETAIL:  Column "age" is a generated column.
```


## Querying the generated column

The generated column works and behaves as a normal column, that is access can be restricted or granted on such column:

```sql
testdb=# REVOKE ALL ON people_gc_stored FROM public;
testdb=# GRANT SELECT( name, age ) ON people_gc_stored TO harry;
```

Since user `harry` has access only on columns `name` and `age`, the user cannot see the dependency column:

```sql
testdb=> SELECT * FROM luca.people_gc_stored LIMIT 5;
ERROR:  permission denied for table people_gc_stored

testdb=> SELECT min( age ), max( age ) FROM luca.people_gc_stored;
 min | max  
-----|------
   1 | 2740
(1 row)

testdb=> SELECT min( birthday ), max( birthday ) FROM luca.people_gc_stored;
ERROR:  permission denied for table people_gc_stored
```


On the other hand, giving access only on `birthday` column does not automatically provide access on `age`:

```sql
testdb=# REVOKE SELECT ON people_gc_stored FROM harry;
testdb=# GRANT SELECT( name, birthday ) ON people_gc_stored TO harry;
```

```sql
testdb=> SELECT min( birthday ), max( birthday ) FROM luca.people_gc_stored;
      min      |    max     
---------------|------------
 0720-12-07 BC | 2019-11-03
(1 row)

testdb=> SELECT min( age ), max( age ) FROM luca.people_gc_stored;
ERROR:  permission denied for table people_gc_stored
```
