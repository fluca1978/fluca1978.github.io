---
layout: post
title:  "A glance at Raku connectivity towards PostgreSQL"
author: Luca Ferrari
tags:
- raku
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A glance at Raku implementation for PostgreSQL database connectivity.

# A glance at Raku connectivity towards PostgreSQL

[Raku](https://raku.org){:target="_blank"} is a great language in my opinion, and I'm using it more and more everyday. I can say it is going to substitute my Perl scripting.
<br/>
<br/>
Raku comes with an extensive module library, that include of course **database connectivity**, that in turn includes features for connecting to PostgreSQL.
<br/>
In this simple article, I'm going to quickly demonstrate how to use a Raku piece of code to do many of the trivial tasks than a database application can do.
<br/>
The script is presented in an incremental way, so the *Connecting to the database* section must be always be as the script preamble.
<br/>
<br/>
The [`DB::Pg`](https://modules.raku.org/dist/DB::Pg:cpan:CTILMES){:target="_blank"} module is somehow similar to Perl 5 `DBD::Pg`, so a lot of concepts and method names will remind the latter.


## Installation

It is possible to use `zef` to install the `DB::Pg` module:

<br/>
<br/>
```raku
% zef install DB::Pg
```
<br/>
<br/>

Depending on the speed of your system and the libraries already installed, it can take a few minutes.
<br/>
<br/>
If you are going to use the `LISTEN`/`NOTIFY` you need to install also the `epoLl`:

<br/>
<br/>
```raku
% zef install epoll
```
<br/>
<br/>


## Connecting to the database

It is now possible to connect to the database using the `DB::Pg` module. For example, a simple script that accepts all parameters (in clear text!) on the command line can be:


<br/>
<br/>
```raku
#!raku

use DB::Pg;

sub MAIN( Str :$host = 'miguel',
          Str :$username = 'luca',
          Str :$password = 'secret',
          Str :$database = 'testdb' ) {

    "Connecting $username @ $host/$database".say;

    my $connection = DB::Pg.new: conninfo => "host=$host user=$username password=$password dbname=$database";

```
<br/>
<br/>


As you can see, the `DB::Pg` module accepts a **conninfo** string.


## Read queries and results

The `.query` method allows for issuing a read query to the database. The result is a `Result` class object, that can be used by means of different methods, most notably with `.hashes` and `.arrays`  that return a sequence of hashes or arrays, one per every row extracted from the query.
<br/>
Special methods like `.rows` and `.columns` provide respectively the number of rows returned by a query and the list of coumn names of the result set.
<br/>
As an example, here it is a simple query:

<br/>
<br/>
```raku
my $query = 'SELECT current_role, current_time';
my $results = $connection.query: $query;

say "The query { $query } returned { $results.rows } rows with columns: { $results.columns.join( ', ' ) }";
for $results.hashes -> $row {
    for $row.kv -> $column, $value {
        say "Column $column = $value";
    }
}
```
<br/>
<br/>

The above piece of code provides an output similar to the following:

<br/>
<br/>
```raku
The query SELECT current_role, current_time returned 1 rows with columns: current_role, current_time
Column current_role = luca
Column current_time = 14:48:47.147983+02
```
<br/>
<br/>


## Cursors

By default, a `.query` method will fetch all the rows from the query, that is a problem with larger datasets. It is possible to use the `.cursor` method that accepts the optional batch size (by default `1000` tuples) and, optionally, the specifier for getting results into a sequence of hashes.
<br/>
As a simple example:


<br/>
<br/>
```raku
for $connection.cursor( 'select * from raku', fetch => 2, :hash ) -> %row {
    say "====================";
    for %row.kv -> $column, $value {
        say "Column [ $column ] = $value";
    }
    say "====================";
}
```
<br/>
<br/>

that produces and output like:

<br/>
<br/>
```raku
====================
Column [ pk ] = 2
Column [ t ] = This is value 0
====================
====================
Column [ pk ] = 3
Column [ t ] = This is value 1
====================
====================
Column [ t ] = This is value 2
Column [ pk ] = 4
====================
====================
Column [ pk ] = 5
Column [ t ] = This is value 3
====================
...
```
<br/>
<br/>


## Write Statements

Write statements can be performed by means of `.execute` method, such as:

<br/>
<br/>
```raku
$connection.execute: q< insert into raku( t ) values( 'Hello World' )>;
```
<br/>
<br/>


## Transactions and Prepared Statements

In order to handle transactions, you need to access the *database handler* that is "masked" into the `DB::Pg` main object. The *database* object provides the method `.begin`, `.rollback`, `.commit` as usual.
<br/>
Moreover, it is possible to use the `.prepare` method to obtained a *prepared statement* that can be cached and used in loops and repetitive tasks. It is worth noting that the `.prepare` method use the `$1`, `$2`, and so on parameter placeholders, and that when a statement accepts a single value it has to be specified without the index in `.execute`.
<br/>
As an example:

<br/>
<br/>
```raku
my $database-handler = $connection.db;
my $statement = $database-handler.prepare: 'insert into raku( t ) values( $1 )';

$database-handler.begin;
$statement.execute( "This is value $_" )  for 0 .. 10;
$database-handler.commit;
$database-handler.finish;
```
<br/>
<br/>

The above loop is equivalent to an SQL transaction like:

<br/>
<br/>
```sql
BEGIN;
INSERT INTO raku( t ) VALUES ('This is value 0' );
INSERT INTO raku( t ) VALUES ('This is value 1' );
INSERT INTO raku( t ) VALUES ('This is value 2' );
...
INSERT INTO raku( t ) VALUES ('This is value 10' );
COMMIT;
```
<br/>
<br/>

The `.finish` method is required because `DB::Pg` handles caching.
Please note that the `.commit` and `.rollback` methods are *fluent*, and return an object instance so that you can call `.commit.finish`.

## Databases vs Connections

Caching is handled so that when a query is issued, a new connection is opened and used. Once the work has completed, the connection is returned to the internal pool. The `DB::Pg::Database` object does the same work of the `DB::Pg` one, with the exception that it does not automatically returns the connection to the pool, so you need to do the `.finish` by yourself.
<br/>
<br/>
Therefore, you can use the same `.query` and `.execute` methods on both the objects, but the `DB::Pg` automatically returns the connection into the internal pool, while the database object allows you for a fine grain control of when to return the connection into the pool.


## Copy

PostgreSQL provides the special `COPY` command, that can be used to copy from and into. There is a method `.copy-in` that executes a `COPY FROM`, while `COPY TO` can be used within an iteration loop:

<br/>
<br/>
```raku
my $file = '/tmp/raku.csv'.IO.open: :w;
for $connection.query: 'COPY raku TO stdout (FORMAT CSV)'  -> $row {
    $file.print: $row;
}
```
<br/>
<br/>

The above exports the CSV result on a text file.
<br/>
To read the data back, it is possible to issue the `.copy-in` method, but you first need to issue an SQL `COPY`. The workflow is:
- issue a `COPY FROM STDIN`;
- use `.copy-data` to slurp all the data;
- use `.copy-end` to notify the database that the `COPY` is concluded.

<br/>
The need for `.copy-end` is an advatange: it is possible to issue different `.copy-data` in a single run, for example to import data from different files.


<br/>
<br/>
```raku
$database-handler = $connection.db;
$database-handler.query: 'COPY raku FROM STDIN (FORMAT CSV)';
$database-handler.copy-data:  '/tmp/raku1.csv'.IO.slurp;
$database-handler.copy-data:  '/tmp/raku2.csv'.IO.slurp;
$database-handler.copy-end;
```
<br/>
<br/>


## Converters

It is possible to specify converters, special *roles* that handle values in and out the database; something that reminds me the *inflate* and *deflate* options of `DBI::Class`.
<br/>
The first step is to add a role to the `converter` instance within the `DB::Pg`, such instance must:
- add a new type conversion;
- add a `convert` method to handle the type stringified value and returns the new value (in any Raku instance).
<br/>
As an example, the following converts a `text` PostgreSQL type into a `Str` Raku object reversed in its content:

<br/>
<br/>
```raku
$connection.converter does role fluca-converter
{
    submethod BUILD { self.add-type( text => Str ) }
    multi method convert( Str:U, Str:D $value) {
        $value.flip.uc;
    }

}

.say for $connection.query( 'select * from raku' ).arrays;
```
<br/>
<br/>

that produces an output similar to:


<br/>
<br/>
```shell
[442 DLROW OLLEH]
[454 DLROW OLLEH]
[466 DLROW OLLEH]
```
<br/>
<br/>

where the string `Hello World` is flipped.


# Listen and Notify

`DB::Pg` can handle also `LISTEN` and `NOTIFY`, and they are able to interact with the `react` dynamic feature of Raku.
<br/>
First of all, create a simple mechanism to notify some events:

<br/>
<br/>
```sql
testdb=> create or replace rule r_raku_insert 
         as on insert to raku 
         do also 
         SELECT pg_notify( 'insert_event', 'INSERTING ROW(S)' );
CREATE RULE

testdb=> create or replace rule r_raku_delete
         as on delete to raku 
         do also 
         SELECT pg_notify( 'delete_event', 'DELETING ROW(S)' );
CREATE RULE


```
<br/>
<br/>


Now it is possible to create a Raku script that waits for incoming events:


<br/>
<br/>
```raku
react {
    whenever $connection.listen( 'delete_event' ) { .say; }
    whenever $connection.listen( 'insert_event' ) { .say; }
}
```
<br/>
<br/>

The aim is that, every time an event is issued, the `.listen` passes the message payload to the `react`` code block. Therefore, issuing some `DELETE` and `INSERT` will result in the output:

<br/>
<br/>
```shell
DELETING ROW(S)
INSERTING ROW(S)
INSERTING ROW(S)

```
<br/>
<br/>

It is possible to stop the listening `react` block with the `.unlisten` method. It is also possible to issue an event via `.notify`.


# Conclusions

The [`DB::Pg`](https://modules.raku.org/dist/DB::Pg:cpan:CTILMES){:target="_blank"} is a great driver for PostgreSQL that allows Raku to exploit a lot of features directly into the language.
