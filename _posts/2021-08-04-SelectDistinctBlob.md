---
layout: post
title:  "Select Distinct Bytea (or Blobs)" 
author: Luca Ferrari
tags:
- oracle
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A strange behaviour I found in Oracle.

# Select Distinct Bytea (or Blobs)

*TLDR: seems to me that PostgreSQL has a more comfortable behaviour than Oracle when dealing with `distinct` and `BLOB`-like fields*

<br/>
<br/>

I'm not an avid Oracle user, at least not as much as I'm with regard to PostgreSQL.
<br/>
In the last days I spot a problem with an application of mine: after having added a `BLOB` column to an Oracle table, a few automated queries began to fail. It was not so simple, in the beginning, to find out what the problem was, but essentially the *ORM* I am using was generating a query with a `distinct` clause, and it seems that Oracle does not accept such kind of query when it involves a `BLOB` or a `CLOB` field.
<br/>
Let's see an example: the `blobby` table is made by a `varchar2` `description` field and a `bdata` field of type `BLOB`.

<br/>
<br/>
```sql
SQL> select distinct bdata, description from blobby;
select distinct bdata, description from blobby
                *
ERROR at line 1:
ORA-00932: inconsistent datatypes: expected - got BLOB

```
<br/>
<br/>
The reported error is somehow obscure to me: **ORA-00932: inconsistent datatypes: expected - got BLOB** does not provide to me enough information to understand what type the system was expecting. However, seeing the `BLOB` final part let me reason about the problem.
<br/>
However, in the begin, I was not even able to reproduce the problem because if you don't specify an explicit column list, the same query works:


<br/>
<br/>
```sql
SQL> select distinct * from blobby;
...
6 rows selected.
```
<br/>
<br/>

I was unable to make the query to work even using a cast to different types, so I guess Oracle cannot handle the query when the columns are explicitly listed. And that was the problem: many ORMs, including the one I'm using, produce queries where all the columns are asked as output fields, and so Oracle was refusing to run the query.


# What About PostgreSQL?

I was curious to see how does PostgreSQL handle the same situation, assuming `BLOB` can be translated into a `bytea` field.

<br/>
<br/>
```sql
testdb=> create table blobby( pk int generated always as identity,
description text, bdata bytea, primary key( pk ) );
testdb=> insert into blobby( description ) select 'Record ' || v from
generate_series( 1, 5 ) v;
INSERT 0 5
testdb=> \lo_import myfile.pdf
lo_import 50626
testdb=> update blobby set bdata = lo_get( 50626 );
UPDATE 5

testdb=> \o test.csv
testdb=> \a
Output format is unaligned.
testdb=> \f ';'
Field separator is ";".
testdb=> select distinct bdata, description from blobby;
-- same as select distinct * from blobby


% ls -1hs test.csv
23M test.csv
```
<br/>
<br/>


Despite the initial part to create and populate the table, as you can see the `SELECT` works both with an explicit column list or a wildcard.


# Conclusions

I don't have any conclusions, and I don't blame a product or another. They just behave differently under pretty much the same context. 
<br/>
I like the PostgreSQL approach the most, it seems more natural. Moreover, Oracle error messages seem to me very obscure!
