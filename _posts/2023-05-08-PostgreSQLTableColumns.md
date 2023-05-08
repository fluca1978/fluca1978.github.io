---
layout: post
title:  "Extracting the list of columns from the catalogs"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A simple look at the PostgreSQL catalogs to get the list of a table's column.

# Extracting the list of columns from the catalogs

The special catalog `pg_attribute` keeps track of every column that your tabular structure holds.
However, before using such catalog, you need to keep in mind some basic rules. In particular, every attribute in the catalog has an ordinality number named `attnum`: when the number is positive the attribute refers to a user defined column, whenever it is negative it represent a PostgreSQL special column.
Moreover, the special column `attisdropped` indicates if the attribute has been dropped. Moreover, when an attribute is dropped, it takes a *special* name in the catalog, like `.......pg.dropped.5......`.


Imagine we create a dummy table as follows:

<br/>
<br/>
```sql
testdb=> CREATE TABLE foo(
a int
, b char
, c int
, d int
, e int
, f int
, g char
, z text
, y char
, k int
, j int
);

testdb=> ALTER TABLE foo DROP COLUMN e;

```
<br/>
<br/>

Getting the list of columns is easy via the catalog **`pg_attribute`**:

<br/>
<br/>
```sql
testdb=> SELECT attname, attnum
FROM pg_attribute
WHERE attrelid = 'foo'::regclass
AND NOT attisdropped;
 attname  | attnum
----------+--------
 tableoid |     -6
 cmax     |     -5
 xmax     |     -4
 cmin     |     -3
 xmin     |     -2
 ctid     |     -1
 a        |      1
 b        |      2
 c        |      3
 d        |      4
 f        |      6
 g        |      7
 z        |      8
 y        |      9
 k        |     10
 j        |     11
(17 rows)

```
<br/>
<br/>

There are few things to note in the above output:
1) the `attnum` is the order the column has been added to the table, in fact it respects the original table definition;
2) if `attnum` is **positive** than the attribute is a *user defined one*, that means it is a column you added to the table;
3) if `attnum` is **negative** than the attribute has been added by the system (i.e., PostgreSQL) for its internal usage;
4) **all attributes listed in `pg_attribute`** can be queried by the user;
5) the dropped column `e` is missing, note how the `attnum` skips the ordering 5.


<br/>

It is now simple enough to get a list of columns and paste it into a query:

<br/>
<br/>
```sql
testdb=> SELECT string_agg( attname, ',' )
FROM pg_attribute
WHERE attrelid = 'foo'::regclass
AND NOT attisdropped;
                       string_agg
---------------------------------------------------------
 a,b,c,cmax,cmin,ctid,d,e,f,g,j,k,tableoid,xmax,xmin,y,z
(1 row)

testdb=> SELECT a,b,c,cmax,cmin,ctid,d,e,f,g,j,k,tableoid,xmax,xmin,y,z FROM foo;
-[ RECORD 1 ]--------
a        |
b        |
c        |
cmax     | 0
cmin     | 0
ctid     | (0,1)
d        |
f        |
g        |
j        |
k        |
tableoid | 44309
xmax     | 0
xmin     | 2075116
y        |
z        | test tuple

```
<br/>
<br/>

Clearly, you can manipulate the query to build something that allows you to choose between user columns and system columns, for example:

<br/>
<br/>
```sql
testdb=> WITH user_columns AS ( SELECT attname FROM pg_attribute
                                WHERE attrelid = 'foo'::regclass AND attnum > 0
								AND NOT attisdropped
								ORDER BY 1 )
, system_columns AS ( SELECT attname FROM pg_attribute
                      WHERE attrelid = 'foo'::regclass AND attnum < 0
					  AND NOT attisdropped
					  ORDER BY 1 )
SELECT string_agg( c.attname, ', ' )
FROM user_columns c;
           string_agg
---------------------------------
 a, b, c, d, f, g, j, k, y, z
(1 row)

```
<br/>
<br/>


You can even build something a little more complex, in order to get for instance the definition of a trigger (or something like that):



<br/>
<br/>
```sql
testdb=> WITH user_columns AS ( SELECT attname FROM pg_attribute
                                WHERE attrelid = 'foo'::regclass AND attnum > 0
								AND NOT attisdropped
								ORDER BY 1 )
, system_columns AS ( SELECT attname FROM pg_attribute
                      WHERE attrelid = 'foo'::regclass AND attnum < 0
					  AND NOT attisdropped
					  ORDER BY 1 )
, user_columns_list AS ( SELECT string_agg( c.attname , ',' ) as l
                         FROM user_columns c )
SELECT 'CREATE TRIGGER tr_foo_ins '
      || ' BEFORE UPDATE OF '
	  || ucl.l
	  || ' ON foo  FOR EACH ROW EXECUTE PROCEDURE f_tr_foo_ins() '
FROM user_columns_LIST ucl;

         ?column?
--------------------------------------------------------------------------------------------------------------------------
 CREATE TRIGGER tr_foo_ins  BEFORE UPDATE OF a,b,c,d,f,g,j,k,y,z ON foo  FOR EACH ROW EXECUTE PROCEDURE f_tr_foo_ins()

```
<br/>
<br/>

This can be pushed into a function or a `EXECUTE` dynamic query to provide a dinamically generated statement.
