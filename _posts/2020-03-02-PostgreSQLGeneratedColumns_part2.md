---
layout: post
title:  "PostgreSQL 12 Generated Columns: another use case"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
When you start realizing how useful can be generated columns, you start using them as part of your workflow.
Here there's another story of mine in the adventures in PostgreSQL-land.

# PostgreSQL 12 Generated Columns: another use case

[I've already written](https://fluca1978.github.io/2019/11/04/PostgreSQL12GeneratedColumns.html){:target="_blank"} about PostgreSQL 12 feature related to [automatically generated columns](https://www.postgresql.org/docs/12/ddl-generated-columns.html){:target="_blank"**.
<br/>
A few days ago I worked on a simple table that contains a single tuple for every file on a filesystem, including the file size and hash. Having the file hash provides a lot of practical analisys, including seeing how many times the file is replicated in the file system.
<br/>
But what if I want to store such duplication information into the table?
<br/>
One solution could be to add a column, and then run a long `UPDATE** to update such column, then insert a trigger to catch every new table modifications.
<br/>
**Or, I can use generated columns!**


## The table structure

The table was structured as follows, and it is quite simple to understand:

```sql
testdb=> \d my_files
                       Table "public.my_files"
  Column   |          Type           | Collation | Nullable | Default 
-----------|-------------------------|-----------|----------|---------
 filename  | character varying(200)  |           |          | 
 directory | character varying(2048) |           |          | 
 md5sum    | character varying(128)  |           |          | 
 bytes     | integer                 |           |          | 
```

The file can be found in the filesystem in the position `directory || filename` (i.e., string concatenation). Every file has its own checksum (`md5sum`) and the size expressed in `bytes`.
<br/>
Please note that this is a de-normalized schema, but it is a simple use case I have to work with so far.
<br/>
The size of the table is quite normal:

```sql
testdb=> SELECT reltuples, relpages, pg_size_pretty( pg_relation_size( 'vace.my_files' ) ) FROM pg_class WHERE relname = 'my_files' AND relkind = 'r';
  reltuples   | relpages | pg_size_pretty 
--------------|----------|----------------
 1.872529e+06 |    40757 | 318 MB
(1 row)
```

## Adding a generated column

Let's add a new column to count the occurrencies of the file, that is how many times the file appears in the filesystem.
<br/>
First of all, a new `IMMUTABLE` function must be generated:

```sql
CREATE FUNCTION f_count_occurrencies( md5sum_to_find text )
RETURNS bigint
AS
    $CODE$
      SELECT count(*)
      FROM   my_files
      WHERE  md5sum = md5sum_to_find;
    $CODE$
LANGUAGE sql
IMMUTABLE;
```

It is a very simple function: it does a `count(*)` of every tuple with a specific checksum.
<br/>
There are two things to note: the function must return a `bigint` because so it does `count()` and, most notably, it must be marked as `IMMUTABLE` because it is what is required to use such function as the engine to compute the generated column values.
<br/>
However, applying such a function did not complete within two hours!

```sql
testdb=> ALTER TABLE my_files 
         ADD COLUMN occurrencies int 
         GENERATED ALWAYS 
         AS ( f_count_occurrencies( md5sum ) ) STORED;


^CCancel request sent
ERROR:  canceling statement due to user request
CONTEXT:  SQL function "f_count_occurrencies" statement 1
Time: 8285400,145 ms (02:18:05,400)
```

Therefore I decided to create an index on the field `md5sum` and try it again:

```sql
testdb=> CREATE INDEX idx_md5sum ON my_files( md5sum );
CREATE INDEX
Time: 3016,571 ms (00:03,017)

testdb=> ALTER TABLE my_files 
         ADD COLUMN occurrencies int 
         GENERATED ALWAYS 
         AS ( f_count_occurrencies( md5sum ) ) STORED;
ALTER TABLE
Time: 120131,809 ms (02:00,132)
```

As you can see, this time it took two minutes to perform the update of the table structure with the automatically computed column, while before the creation of the index it had not finished within two hours.

The table does not occupy much more space than before:

```sql
testdb=> SELECT reltuples, relpages, pg_size_pretty( pg_relation_size( 'my_files' ) ) FROM pg_class WHERE relname = 'my_files' AND relkind = 'r';
  reltuples   | relpages | pg_size_pretty 
--------------|----------|----------------
 1.872529e+06 |    41492 | 324 MB
(1 row)

```

so with six extra megabytes we have now the information replicated on every row. The table increased of around `1.8%` in size but make now computation about how much a file is replicated is straightforward.


## A generated column cannot be based on a generate column

Once you get used to generated column, you simply want more.
<br/>
Making a column that indicates, per file, how much disk space it consumes due to its replicated version seems easy, but it is a little tricky. **A generated column cannot be based on another generated column**, it would be a circular dependency or better, a dependency that PostgreSQL cannot solve (there should be a generation order and sooner or later you could end up with a circular dependency).
<br/>
This means we cannot exploit the `occurrencies` column in the count of the disk space. In fact, let's add a generated column based on that, so first let's create a simple computation function:

```sql
CREATE FUNCTION f_compute_duplicated_size( bytes int, how_many_times int )
    RETURNS int
    AS $CODE$
      SELECT bytes * how_many_times;
    $CODE$
    LANGUAGE sql IMMUTABLE;

```

Note that the function exploits the generated column `occurrencies`, that is *we are generating a column on the basis of another generated column*, something PostgreSQL will avoid and in fact:


```sql
testdb=> ALTER TABLE my_files 
         ADD COLUMN duplication_bytes int 
         GENERATED ALWAYS 
         AS ( f_compute_duplicated_size( bytes, occurrencies ) ) STORED;
ERROR:  cannot use generated column "occurrencies" in column generation expression
DETAIL:  A generated column cannot reference another generated column.
Time: 17,900 ms
```

We need a trick to make the generated column indipendent from the other already generated one. Therefore, we can use a function like the following, that does not exploit any generated column:

```sql
CREATE FUNCTION f_compute_duplicated_size( md5sum_to_find text )
RETURNS bigint
AS $CODE$
    SELECT sum( bytes)
    FROM   my_files
    WHERE  md5sum = md5sum_to_find;
$CODE$
LANGUAGE sql IMMUTABLE;
```

and add the column, this time with success:

```sql
testdb=> ALTER TABLE my_files 
         ADD COLUMN duplication_bytes int 
         GENERATED ALWAYS 
         AS ( f_compute_duplicated_size( md5sum ) ) STORED;
ALTER TABLE
Time: 119696,310 ms (01:59,696)
```

Again, we are exploiting the `md5sum` index to keep the table modification at a rational speed.


## More columns...more columns quick!

You get the point, it is now possible to enhance the table to get much more generated columns.
Of course, the risk is to denomarlize the data more and more, so using this approach depends on what is your aim. Since I'm using this table as a workbench to make some simulations on a filesystem, I don't care too much about the normalization of the data, rather I do care to be able to do simple queries and get the result.
<br/>
So what's next?
<br/>
Let's add a column to decide the file *MIME type* based on a very poor approach: the file extension!
Of course, I do trust my filenames to be correct with respect to the relationship between the extension and the MIME type.
<br/>
The approach is always the same:
1) create an immutable function;
2) alter the table.

<br/>
The function I use exploits the regular expression engine:

```sql
CREATE FUNCTION f_compute_file_type( filename text )
RETURNS text
AS $CODE$
    SELECT  upper( ( regexp_match( trim( filename ),
                     '\.([a-zA-Z0-9]{3,4})$' ) )[ 1 ] );
$CODE$
LANGUAGE SQL IMMUTABLE;
```

The function returns the very last extension assuming it could be three or four characters (or digits).
Then adding the column is boring:

```sql
testdb=> ALTER TABLE my_files 
         ADD COLUMN filetype text 
         GENERATED ALWAYS 
         AS ( f_compute_file_type( filename ) ) STORED;
ALTER TABLE
Time: 21452,153 ms (00:21,452)
```

## What is the final effect on the table?

We added three generated columns in the table, what is the impact about the space?
<br/>

```sql
testdb=> SELECT reltuples, relpages, pg_size_pretty( pg_relation_size( 'my_files' ) ) FROM pg_class WHERE relname = 'my_files' AND relkind = 'r';
  reltuples   | relpages | pg_size_pretty 
--------------|----------|----------------
 1.872529e+06 |    43704 | 341 MB
(1 row)
```

Therefore the table size has grown from `318 MB` to `341 MB`, meaning `7%`. We have now a bigger table, to some extent de-normalized, but with a lot of more data to be used for analysis. Moreover, we can drop the index on `md5sum` since we could not need it anymore.


## Generated Columns are not fixed!

Well, this could sound trivial, but the fact is that a generated column is not a *compute-once* column: the value of the column is updated every time its dependending-on columns are modified.
<br/>
In the previous example you have seen that the `filetype` column depends on the `filename` one, but if we change the name to the file, for example because we don't care anymore about the file extension, we are going to mess up also the `filetype` column.
<br/>
The rule of thumb therefore is: if you need *computed-once* data use a materialized view (or a fixed column in the table), otherwise you can use generated columns.
