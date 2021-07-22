---
layout: post
title:  "PostgreSQL Extension Catalogs" 
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to see the available and/or installed extensions?

# PostgreSQL Extension Catalogs

There are three main catalogs that can be useful when dealing with extensions:
- [`pg_extension`](https://www.postgresql.org/docs/current/catalog-pg-extension.html){:target="_blank"};
- [`pg_available_extension_versions`](https://www.postgresql.org/docs/13/view-pg-available-extension-versions.html){:target="_blank"};
- [`pg_available_extensions`](https://www.postgresql.org/docs/13/view-pg-available-extensions.html){:target="_blank"}.

<br/>
The former one, `pg_extension` provides information about *which extensions are installed in the current database*, while the latter, `pg_available_extensions` provides information about *which extensions are available to the cluster*.
<br/>
The difference is simple: to be used an extension must appear first on `pg_available_extensions`, that means it has been installed on the cluster (e.g., via `pgxnclient`). From this point on, the extension can be *installed* into the database by means of a `CREATE EXTENSION` statement; as a result the extension will appear into the `pg_extension` catalog.

<br/>
As an example:

<br/>
<br/>
```sql
testdb=> select name, default_version from pg_available_extensions;
        name        | default_version 
--------------------|-----------------
 intagg             | 1.1
 plpgsql            | 1.0
 dict_int           | 1.0
 dict_xsyn          | 1.0
 adminpack          | 2.1
 intarray           | 1.3
 amcheck            | 1.2
 autoinc            | 1.0
 isn                | 1.2
 bloom              | 1.0
 fuzzystrmatch      | 1.1
 jsonb_plperl       | 1.0
 btree_gin          | 1.3
 jsonb_plperlu      | 1.0
 btree_gist         | 1.5
 hstore             | 1.7
 hstore_plperl      | 1.0
 hstore_plperlu     | 1.0
 citext             | 1.6
 lo                 | 1.1
 ltree              | 1.2
 cube               | 1.4
 insert_username    | 1.0
 moddatetime        | 1.0
 dblink             | 1.2
 earthdistance      | 1.1
 file_fdw           | 1.0
 pageinspect        | 1.8
 pg_buffercache     | 1.3
 pg_freespacemap    | 1.2
 pg_prewarm         | 1.2
 pg_stat_statements | 1.8
 pg_trgm            | 1.5
 pg_visibility      | 1.2
 pgcrypto           | 1.3
 pgrowlocks         | 1.2
 pgstattuple        | 1.5
 postgres_fdw       | 1.0
 refint             | 1.0
 seg                | 1.3
 bool_plperl        | 1.0
 plperlu            | 1.0
 sslinfo            | 1.2
 anon               | 0.9.0
 tablefunc          | 1.0
 tcn                | 1.0
 tsm_system_rows    | 1.0
 bool_plperlu       | 1.0
 tsm_system_time    | 1.0
 pgaudit            | 1.5
 pg_qualstats       | 2.0.2
 unaccent           | 1.1
 plperl             | 1.0
 orafce             | 3.13
 uuid-ossp          | 1.1
 xml2               | 1.1
 pg_background      | 1.0

```
<br/>
<br/>

The above list represents all the available extensions installed on the cluster, thus those I can execute a `CREATE EXTENSION` against.
<br/>
<br/>
The `pg_available_extensions` has an `installed_version` field that provides the version number of the extension installed in the current database, or `NULL` if the extension is not installed in the current database. Therefore, in order to know if an extension is installed or not in a database, you can run a query like the following:


<br/<
<br/>
```sql
testdb=> select name, default_version, installed_version 
         from pg_available_extensions 
         where installed_version is not null;
     name      | default_version | installed_version 
---------------|-----------------|-------------------
 plpgsql       | 1.0             | 1.0
 dblink        | 1.2             | 1.2
 orafce        | 3.13            | 3.13
 pg_background | 1.0             | 1.0

```
<br/>
<br/>
This is a little too much effort, and since extension could have been installed with different *flags* in different database, the `pg_extension` catalog provides a more detailed and narrowed information: it lists all extensions that have been installed on the current database.

<br/>
Therefore, to see what a database can use, that means which extensions it has access to, I need to use the `pg_extension` catalog:

<br/>
<br/>
```sql
testdb=> select extname, extversion from pg_extension ;
    extname    | extversion 
---------------|------------
 plpgsql       | 1.0
 orafce        | 3.13
 dblink        | 1.2
 pg_background | 1.0

```
<br/>
<br/>

The current database has a much smaller list of available extensions.


## Extension Version Numbers

As you know, an extension can come with different version number and the beauty of this mechanism is that it is easy to upgrade an extension from one version to another.
<br/>
The `pg_available_extensions` catalog provides only the last (i.e., newest) version of an available extension. Let's try with a very popular extension: `pg_stat_statements`:

<br/>
<br/>
```sql
testdb=> select name, default_version, installed_version
         from pg_available_extensions 
         where name = 'pg_stat_statements';
        name        | default_version | installed_version 
--------------------|-----------------|-------------------
 pg_stat_statements | 1.8             | 

```
<br/>
<br/>
The extension could be installed to the version `1.8` and is currently not available in the current database.
<br/>
But what about other version numbers?
<br/>
The catalog `pg_available_extension_versions` provides a list of all available versions an extension is currently available:


<br/>
<br/>
```sql
testdb=> select name, version, installed, relocatable
         from pg_available_extension_versions 
         where name = 'pg_stat_statements'
         order by version;
        name        | version | installed | relocatable 
--------------------|---------|-----------|-------------
 pg_stat_statements | 1.4     | f         | t
 pg_stat_statements | 1.5     | f         | t
 pg_stat_statements | 1.6     | f         | t
 pg_stat_statements | 1.7     | f         | t
 pg_stat_statements | 1.8     | f         | t

``` 
<br/>
<br/>
As you can see, the extension is available in five different versions, and I can choose the version that fit the best my requirements.
<br/>
This catalog provides different information, in particular it can give you an idea if the extension can be installed only by superusers (field `superuser`) or by a user with appropriate privileges (field `trusted`), as well as other required extensions (field `requires_name`), and relocatability.
