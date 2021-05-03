---
layout: post
title:  "pg_dump and inserts"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-conf
permalink: /:year/:month/:day/:title.html
---
pg_dump supports a few useful options to export data as a list of INSERTs


# pg_dump and inserts

`pg_dump(1)` is the default tool for doing backups of a PostgreSQL database.
<br/>
I often got answers about how to produce a **more portable** output of the database dump, with *portable* meaning truly *"loadable into another PostgreSQL version or even a different database"*.
<br/>
In fact, `pg_dump` defaults to use `COPY` for bulkd loading data:

<br/>
<br/>
```shell
% pg_dump -a  -t wa -U luca testdb 
...
COPY luca.wa (pk, t) FROM stdin;
9200673 Record #1
9200674 Record #2
9200675 Record #3
9200676 Record #4
9200677 Record #5
9200678 Record #6
9200679 Record #7
9200680 Record #8
9200681 Record #9
...
```
<br/>
<br/>
As you can guess, `COPY` is usable only in PostgreSQL and not into other database. So, how to handle a text dump that can be used into other databases?
<br/>
No need to worry: `pg_dump` has a few features to handle such need.
<br/>
In particular, the following options can be useful:
- `--inserts` removes the `COPY` and substitutes it with `INSERT` statements, one per tuple;
- `--column-inserts` similar to the previous, but each `INSERT` has the list of named columns;
- `rows-per-inserts` a number of tuples a single `INSERT` statement can handle, useful for a better bulk loading (but could be less portable).

<br/>
There are also some other useful options:
- `--quote-all-identifiers` force the quoting of the identifiers, and this is useful when preparing data for a different database;
- `--use-set-session-authorization` when dealing with ownership of objects, use SQL standard commands;`
- `--no-comments`, this is not a very "technical" aspect, but when you are going to load your dump into another database you probably do not want to import comments since they could be handled differently. Similarly, there are other `--no` options that are specific to PostgreSQL, like `--no-publications` to avoid replicating publications, and so on.

<br/>
In the following I will use the same example table `wa` table with just two columns and a bunch of records, so that you can easily compare the output differences.

## Defaulting to `INSERT`

In order to better understand the difference between every single option, let's see a few examples:
<br/>
<br/>
```shell
% pg_dump -a  -t wa  --inserts -U luca testdb 
...
INSERT INTO luca.wa VALUES (9200673, 'Record #1');
INSERT INTO luca.wa VALUES (9200674, 'Record #2');
INSERT INTO luca.wa VALUES (9200675, 'Record #3');
INSERT INTO luca.wa VALUES (9200676, 'Record #4');
INSERT INTO luca.wa VALUES (9200677, 'Record #5');
INSERT INTO luca.wa VALUES (9200678, 'Record #6');
INSERT INTO luca.wa VALUES (9200679, 'Record #7');
...
```
<br/>
<br/>

As you can see from the above, the `COPY` has been translated into a set of `INSERT`s. This of course has the drawback of having a slower buk loading.
<br/>
Just to do another example, let's see how it does change the output with identifier quotiong:
<br/>
<br/>
```shell
% pg_dump -a  -t wa  --inserts --quote-all-identifiers -U luca testdb 
...
INSERT INTO "luca"."wa" VALUES (9200673, 'Record #1');
INSERT INTO "luca"."wa" VALUES (9200674, 'Record #2');
INSERT INTO "luca"."wa" VALUES (9200675, 'Record #3');
INSERT INTO "luca"."wa" VALUES (9200676, 'Record #4');
INSERT INTO "luca"."wa" VALUES (9200677, 'Record #5');
INSERT INTO "luca"."wa" VALUES (9200678, 'Record #6');
...
```
<br/>
<br/>
And the table and schema name has been quoted.
<br/>
What if you want also the column list on every `INSERT`? The optin ``--column-inserts` is there to explode the list of columns:`
<br/>
<br/>
```shell
% pg_dump -a  -t wa  --column-inserts --quote-all-identifiers -U luca testdb 
...
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200673, 'Record #1');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200674, 'Record #2');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200675, 'Record #3');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200676, 'Record #4');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200677, 'Record #5');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200678, 'Record #6');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200679, 'Record #7');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200680, 'Record #8');
INSERT INTO "luca"."wa" ("pk", "t") VALUES (9200681, 'Record #9');
...
```
<br/>
<br/>

despite the usage or not of the `--quote-all-identifiers`, each `INSERT` has the list of the columns the values are referring to.
<br/>
The last case, a middle path between the `COPY` and a single `INSERT` per tuple, is the `--rows-per-insert` that allows you specify the maximum number of rows every `INSERT` will handle:
<br/>
<br/>
```shell
% pg_dump -a  -t wa  --rows-per-insert=3 --quote-all-identifiers -U luca testdb 
...
INSERT INTO "luca"."wa" VALUES
        (9200688, 'Record #16'),
        (9200689, 'Record #17'),
        (9200690, 'Record #18');
INSERT INTO "luca"."wa" VALUES
        (9200691, 'Record #19'),
        (9200692, 'Record #20');
...
```
<br/>
<br/>
Note how the last `INSERT` has only two tuples instead of the specified `3`: the `pg_dump` is smart enough to let your `INSERT` to not loose a single row, so if there is not enough data left, the `INSERT` involves less rows.


## Avoid `ALTER TBALE` to set ownership

If the dump includes the table data structure, `pg_dump` will issue appropriate commands to change the ownership. For example:


<br/>
<br/>
```shell
% pg_dump -C  -t wa  -U luca testdb
...
CREATE TABLE luca.wa (
    pk integer NOT NULL,
    t text
);

ALTER TABLE "luca"."wraparaound_pk_seq" OWNER TO "luca";
...
```
<br/>
<br/>
The option `--use-set-session-authorization` produces a more portable SQL output:

<br/>
<br/>
```shell
% pg_dump -C  -t wa --use-set-session-authorization -U luca testdb
...
SET SESSION AUTHORIZATION 'luca';


CREATE TABLE luca.wa (
    pk integer NOT NULL,
    t text
);

...
```
<br/>
<br/>

As you can see, the user is *set* in the beginning, so that automatically all created objects will belong to such user.
