---
layout: post
title:  "'generated always as identity' columns do not have default values (or do they?)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perl
permalink: /:year/:month/:day/:title.html
---
Something strange I discovered while using `DBIx::Class` and `DBI`.

# 'generated always as identity' columns do not have default values (or do they?)

PostgreSQL has two ways of defining what other databases call *an auto-increment column*:
- `serial`
- `generated always as identity`

The former, `serial`, is the oldest way of declaring an auto-increment column: it creates a sequence and attaches the default value of the column to the `nextval()` of the sequence.
The latter, `generated always as identity`, is  the *newest* (even if not so new!) **declarative** way of doing the same stuff as `serial` does: it creates a sequence and attaches the sequence and the table column.

So what is the difference?

In short, with `serial` you get two independent objects (a column and a sequence) that behave separatly, even if the value of the column is tied to the next value of the sequence. If there is the need to restart the counter, the table does not know anything, so you need to explicitly work against the sequence.
On the other hand, with `generated always as identity`, the table column *knows* the sequence used to populate itself, and therefore it is possible to reset the sequence by working against the table. So for instance, you can do a `ALTER TABLE foo ALTER COLUMN pk RESTART;` without having to know the sequence name behind the `pk` column.

Usually, I do explain that the `generated always as identity` is the best way to proceed, because it gives all the advantages of `serial` with a more declarative way of handling special cases.

**So, you should use `generated always as identity`, except when you should not!**


## The begin of the problems: `dbicdump` and `DBIx::Schema::Loader`

I was migrating a schema from SQLite3 to our beloved database, so I converted every SQLite3 `autoincrement` column to `int generated always as identity`.
So far, so good, simple enough.

Then I used Perl `DBIx::Schema::Loader` and `dbicdump` to dump the database structure into so called *schema*, with objects to use.

And last, I used the generated objects. And here it is where problems become...

`DBIx::Class` was complaining about my *auto-increment* columns not being defined as such. What was wrong?

I inspected a class at glance, and I got the following:


<br/>
<br/>
```perl
__PACKAGE__->add_columns(
  "pk",
  {
    data_type         => "integer",
    is_nullable       => 0,
  },
...
```
<br/>
<br/>


compared to the SQlite3 definition

<br/>
<br/>
```perl
__PACKAGE__->add_columns(
  "pk",
  {
    data_type         => "integer",
    is_auto_increment => 1,
    is_nullable       => 0,
  },
...
```
<br/>
<br/>

immediatly revelead that there was a missing `is_auto_increment` variable definition for the same column.

**`DBIx::Schema::Loader` was not understanding the column definition, at least not as I was expecting.**


## Investigating the problem

I decided to create a simple table with the two possible column types, and dump the schema to see what happens.

With a PostgreSQL table defined as:

<br/>
<br/>
```sql
testdb=> \d foo
                             Table "public.foo"
 Column |  Type   | Collation | Nullable |             Default
--------+---------+-----------+----------+---------------------------------
 pk     | integer |           | not null | generated always as identity
 kp     | integer |           | not null | nextval('foo_kp_seq'::regclass)

```
<br/>
<br/>

the resulted output from `dbicudmp` is:

<br/>
<br/>
```perl
__PACKAGE__->add_columns(
  "pk",
  {
    data_type         => "integer",
    is_nullable       => 0,
  },
  "kp",
  {
    data_type         => "integer",
    is_auto_increment => 1,
    is_nullable       => 0,
    sequence          => "author_kp_seq",
  },
  ...
```
<br/>
<br/>

It is clear that **`dbicdump` is able to understand `serial` columns, while it is not able to understand the default value of `generated always as identity`**!


## More investigation: `DBIx::Class::Schema::Loader::DBI::Pg`

I was puzzled about the problem, so I decided to try to dig about how `DBIx::Schema::Loader` understands the definition of PostgreSQL table columns.
It turned out, that `DBIx::Class::Schema::Loader::DBI::Pg` is the specific driver behind how the loader interacts with PostgreSQL meta information.

In particular, the `_columns_info_for` function tries to get the metadata for every column of the table, and in particular in such function you can find a piece of code like the following:

<br/>
<br/>
```perl
 # process SERIAL columns
 if ( ${ $info->{default_value} } =~ /\bnextval\('([^:]+)'/i ) {
   $info->{is_auto_increment} = 1;
   $info->{sequence}          = $1;
   delete $info->{default_value};
 }

```
<br/>
<br/>


Despite the comment, it is clear that the branch is evaluating the fact that the `default_value` must be like `nextval`.
*There are no other places that handle the `autoincrement` and sequence* in the method.

But what is that `$info` hash? It is coming from `DBI::column_info`.


## More and more investigation: `DBI::column_info`

The `DBI` driver interface provides a `column_info` method that provides an hash with a lot of useful information about the column definition.

It is really simple to write a dummy Perl program to dump the structure of the `foo` table presented before:

<br/>
<br/>
```perl
use v5.38;

use DBI;

my $db = DBI->connect( 'dbi:Pg:dbname=testdb;host=venkman;port=5432',
		       q/luca/,
		       q/XXXXXXXXXXX/ );


# testdb=> \d foo
#                             Table "public.foo"
#  Column |  Type   | Collation | Nullable |             Default
# --------+---------+-----------+----------+---------------------------------
#  pk     | integer |           | not null | generated always as identity
#  kp     | integer |           | not null | nextval('foo_kp_seq'::regclass)


my $statement = $db->column_info( undef, q/public/, q/foo/, q/pk/ );

while ( my $row = $statement->fetchrow_hashref ) {
    use Data::Dumper;
    say Dumper( $row );
}




say "==================================";
$statement = $db->column_info( undef, q/public/, q/foo/, q/kp/ );
while ( my $row = $statement->fetchrow_hashref ) {

    use Data::Dumper;
    say Dumper( $row );
}

```
<br/>
<br/>


The program produces the following (trimmed) output:

<br/>
<br/>
```shell
% perl ~/tmp/test.pl
$VAR1 = {
          'TYPE_NAME' => 'integer',
          'pg_schema' => 'public',
          'pg_type' => 'integer',
          'NULLABLE' => 0,
          'COLUMN_DEF' => undef,
          'IS_NULLABLE' => 'NO',
          'pg_column' => 'pk',
          'COLUMN_NAME' => 'pk',
          'pg_table' => 'foo',
          'TABLE_NAME' => 'foo',
          'TABLE_SCHEM' => 'public',
          'pg_constraint' => undef
		  ...
        };

==================================
$VAR1 = {
          'pg_column' => 'kp',
          'COLUMN_NAME' => 'kp',
          'DECIMAL_DIGITS' => undef,
          'pg_table' => 'foo',
          'TABLE_NAME' => 'foo',
          'TABLE_SCHEM' => 'public',
          'TYPE_NAME' => 'integer',
          'pg_type' => 'integer',
          'NULLABLE' => 0,
          'COLUMN_DEF' => 'nextval(\'foo_kp_seq\'::regclass)',
          ...
        };

```
<br/>
<br/>

The interesting part is **`COLUMN_DEF`** that defines the default value of the column: note how in the case of `serial` there is the `nextval()` call, while in the case of `generated always as identity` there is nothing. This is the problem.


## Don't blame `DBI`!

This is not a bug of `DBI` in a strict sense, so don't blame the Perl Database Interface!

In fact, PostgreSQL is a little tricky about giving back information about `generated always as identity` columns:

<br/>
<br/>
```sql
testdb=> select a.attname, a.attidentity,
		(select pg_get_expr( d.adbin, d.adrelid, true )
		from pg_attrdef d
		where d.adrelid = a.attrelid
		and d.adnum = a.attnum )
		from pg_attribute a
		where a.attrelid = 'foo'::regclass
		and a.attname in ( 'pk', 'kp' );

 attname | attidentity |           pg_get_expr
---------+-------------+---------------------------------
 kp      |             | nextval('foo_kp_seq'::regclass)
 pk      | a           |
(2 rows)

```
<br/>
<br/>


As you can see, the only thing it is possible to extract is the fact that the column has been defined as an identity one (`attidentity = a `).


## How does `DBD::Pg` finds out the information about a column?

It turned out that `DBD::Pg` is using [a very long query to get out the information about a column](https://github.com/bucardo/dbdpg/blob/master/Pg.pm#L501){:target="_blank"}.  I've opened a [ticket on DBB::Pg](https://github.com/bucardo/dbdpg/issues/123){:target="_blank"}.


# Conclusions

I never thought about the possible difficulty in introspecting a generated column as identity. While I'm still convinced about the fact that it is better to use such columns instead of `serial`, the introspective frameworks could gain more information from the `serial` attribute defintions.
