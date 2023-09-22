---
layout: post
title:  "SQLite3: Watch out for missing Group By!"
author: Luca Ferrari
tags:
- sqlite
permalink: /:year/:month/:day/:title.html
---
SQLite3 intentionally allow you to use aggregate functions without `GROUP BY` clauses. This can lead to strange errors.

# SQLite3: Watch out for missing Group By!

SQLite3 intentionally allows you to use aggregate functions even without a `GROUP BY` clause in your queries. This can lead to some strange errors and behaviors, since the database is not going to raise an error, rather it will produce a *almost valid tuple*, but **only one tuple** (because you are using an aggregation) instead of all the expected rows.


## An examle scenario

Consider an *author*/*book* database, where every author can author more than one book. Therefore, we have an `author` table that joins the `book` table with a join table named `j_author_book`. The following is the excerpt to create the database:

<br/>
<br/>
```sql
% sqlite3 test.db
SQLite version 3.40.1 2022-12-28 14:03:47
Enter ".help" for usage hints.

sqlite> CREATE TABLE author( name varchar(255), pk integer primary key autoincrement );

sqlite> CREATE TABLE book( title varchar(255), pk integer primary key autoincrement );

sqlite> CREATE TABLE j_author_book( author integer, book integer, foreign key (author) references author(pk), foreign key (book) references book(pk ) );

sqlite> INSERT INTO author( name ) VALUES( 'LUCA' );
sqlite> INSERT INTO author( name ) VALUES( 'ENRICO' );
sqlite> INSERT INTO book( title ) VALUES( 'Learn PostgreSQL' );
sqlite> INSERT INTO book( title ) VALUES( 'PostgreSQL 11 Server Side Programming' );
sqlite> INSERT INTO book( title ) VALUES( 'PostgreSQL High Performances' );
sqlite> INSERT INTO j_author_book VALUES( 1, 1 );
sqlite> INSERT INTO j_author_book VALUES( 1, 2 );
sqlite> INSERT INTO j_author_book VALUES( 2, 1 );
sqlite> INSERT INTO j_author_book VALUES( 2, 3 );

sqlite> SELECT * FROM author;
LUCA|1
ENRICO|2

sqlite> SELECT * FROM book;
Learn PostgreSQL|1
PostgreSQL 11 Server Side Programming|2
PostgreSQL High Performances|3

sqlite> SELECT * FROM j_author_book ;
1|1
1|2
2|1
2|3
sqlite> SELECT a.name, b.title FROM author a JOIN j_author_book j ON a.pk = j.author JOIN book b ON b.pk = j.book;
LUCA|Learn PostgreSQL
LUCA|PostgreSQL 11 Server Side Programming
ENRICO|Learn PostgreSQL

```
<br/>
<br/>


As you can see, the author `LUCA` has two books, and the same happens for the author `ENRICO` but the two authors co-authored only a single book together.

## See the problem in action

With the above example database, let's see the problem in action.
How many books have every author written?

<br/>
<br/>
```sql
sqlite> SELECT a.name, count(*) FROM author a JOIN j_author_book j ON a.pk = j.author;
LUCA|4
```
<br/>
<br/>


What? `LUCA` has written only two books, not four!
Please note that the resulting number is the total number of entries in the table `j_author_book`.

<br/>

What is the problem? **The query is broken: it is missing a `GROUP BY` clause!**

<br/>
<br/>
```sql
sqlite> SELECT a.name, count(*) FROM author a JOIN j_author_book j ON a.pk = j.author GROUP BY a.name;
ENRICO|2
LUCA|2


```
<br/>
<br/>

This produces the right result, as expected.


## More on the problem: **it's not a bug!**

The above behavior of SQLite is not a bug, rather a [documented feature](https://www.sqlite.org/quirks.html#aggregate_queries_can_contain_non_aggregate_result_columns_that_are_not_in_the_group_by_clause){:target="_blank"}.

While I don't agree with the feature itself, since I **prefer to adhere to SQL standard**, the documentation explains that this feature of SQLite has a few edge cases. To understand these edge cases, consider a modification to the `book` table to include the publication year:

<br/>
<br/>
```sql

sqlite> ALTER TABLE book ADD COLUMN publication_year int;


sqlite> UPDATE book SET publication_year = 2018 WHERE pk = 2;
sqlite> UPDATE book SET publication_year = 2017 WHERE pk = 3;
sqlite> UPDATE book SET publication_year = 2020 WHERE pk = 1;


```
<br/>
<br/>

Imagine we want to find out which author published a book as first, and one as last. In this case, we don't need the `GROUP BY` clause (*only in SQLite3!*:

<br/>
<br/>
```sql
sqlite> SELECT a.name, min( publication_year ) FROM author a JOIN j_author_book j ON j.author = a.pk JOIN book b ON b.pk = j.book;
ENRICO|2017

sqlite> SELECT a.name, max( publication_year ) FROM author a JOIN j_author_book j ON j.author = a.pk JOIN book b ON b.pk = j.book;
LUCA|2020

```
<br/>
<br/>

While this can simplify the usage of some aggregation functions, I strongly believe that having porability and adherence to the SQL standard is much more useful for these edge cases.



# Conclusions

SQLite3 is a great database, but you have to know its own non-standard behaviors. I'm not saying such features are absolutely a bad idea, but not having the database to raise exceptions against non-standard queries, could lead an untested application to deploy errors and get back wrong data.
