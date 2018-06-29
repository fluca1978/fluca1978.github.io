---
layout: post
title:  "PostgreSQL Extended Statistics"
author: Luca Ferrari
tags:
- postgresql
- itpug
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PostgreSQL 10 allows users to define extended statistics to help the planner understand data dependencies.

# PostgreSQL Extended Statistics

PostgreSQL 10 defines a set of *extented statistics*, mainly for intra-column dependencies and distinct values.
[New commands](https://www.postgresql.org/docs/10/static/sql-createstatistics.html) have been added to create and rop such extended statistics, and this post just covers the surface of this new feature directly as long as I was experimenting with that (and as usual, comments are welcome!).

## A sample data set

Assume you have a table defined as follows:

```sql
CREATE TABLE expenses(
   pk int GENERATED ALWAYS AS IDENTITY,
   value money,      
   day date,
   quarter int,
   year int,
   incoming boolean DEFAULT false, 
   account  text DEFAULT 'cash',
   PRIMARY KEY ( pk ),
   CHECK( value <> 0::money ),
   CHECK( quarter = EXTRACT( quarter FROM day ) ),
   CHECK( year = EXTRACT( year FROM day ) )
);
```

clearly there are a few dependencies:
- column `quarter` depends on the value of column `day`;
- column `year` depends on the value of `day` too,
- column `incoming` is `true` when the `value` is greater than zero, `false` otherwise.

And since there are these dependencies, there must be something that ensures us the columns *move* together, so for instance there is a simple trigger as follows:

```sql
CREATE OR REPLACE FUNCTION f_expenses()
RETURNS TRIGGER
AS $code$
BEGIN
  IF NEW.day IS NULL THEN
     NEW.day := CURRENT_DATE;
  END IF;
  
  IF NEW.value = 0::money THEN
     NEW.value := 0.01::money;
  END IF;

  NEW.quarter  := EXTRACT( quarter FROM NEW.day );
  NEW.year     := EXTRACT( year FROM NEW.day );
  IF NEW.value > 0::money THEN
     NEW.incoming := true;
  ELSE
    NEW.incoming := false;
  END IF;

  RETURN NEW;
END
$code$
LANGUAGE plpgsql;



CREATE TRIGGER tr_expenses BEFORE INSERT OR UPDATE ON expenses
FOR EACH ROW
EXECUTE PROCEDURE f_expenses();
```

In a /normal/ situation the planner cannot know such dependencies and, consequently, cannot take advantage of them.
In order to see how this can change in PostgreSQL 10, let's first insert some records:

```sql
WITH v(value, counter, account) AS (
   SELECT ( random() * 100 )::numeric + 1 * CASE v % 2 WHEN 0 THEN 1 ELSE -1 END,
          v,
          CASE v % 3 WHEN 0 THEN 'credit card'
                     WHEN 1 THEN 'bank'
                     ELSE 'cash'
         END
  FROM generate_series( 1, 2000000 ) v
)  
          
INSERT INTO expenses( value, day, account )
SELECT value, CURRENT_DATE - ( counter % 1000 ), account
FROM v;
```

which inserts 2 millions rows (roughly 130 MB of data), 2000 tuples per day.
It is possible to check how many values are there, for instance in the year 2016:

```sql
SELECT                                     
 count(*) FILTER( WHERE year = 2016 ) AS total_2016_by_year, 
 count(*) FILTER( WHERE EXTRACT( year FROM day ) = 2016 ) AS total_2016_by_day,
 count(*) FILTER( WHERE year = 2016 AND incoming = true AND day = '2016-7-19'::date ) AS incoming_by_year, 
 count(*) FILTER( WHERE EXTRACT( year FROM day ) = 2016 AND incoming = true AND day = '2016-7-19'::date ) AS incoming_2016_by_day,
 pg_size_pretty( pg_relation_size( 'expenses' ) ) AS table_size
 FROM expenses;

total_2016_by_year   | 732000
total_2016_by_day    | 732000
incoming_by_year     | 1982
incoming_2016_by_day | 1982
table_size           | 136 MB
```



## Without the extended statistics

What happens when we search for all the expenses of a particular year?

```sql
EXPLAIN  
SELECT *
FROM expenses
WHERE incoming = true
AND   value > 0.0::money
AND   day = '2016-7-19'::date
AND   year = 2016;
                                               QUERY PLAN                                               
--------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..35128.37 rows=697 width=32)
   Workers Planned: 2
   ->  Parallel Seq Scan on expenses  (cost=0.00..34058.67 rows=290 width=32)
         Filter: (incoming AND (day = '2016-07-19'::date) AND (year = 2016) AND (value > (0.0)::money))

```

The planner estimates *697* tuples while we know that the above filtering predicates will provide *1982* tuples.


## With the extended statistics

Let's inform the system about the column dependencies:

```sql
CREATE STATISTICS stat_day_year ( dependencies )
 ON day, year
 FROM expenses;

CREATE STATISTICS stat_day_quarter ( dependencies )
 ON day, quarter
 FROM expenses;
 
CREATE STATISTICS stat_value_incoming( dependencies )
 ON value, incoming
 FROM expenses;
```

There is a special view [`pg_statistic_ext`](https://www.postgresql.org/docs/10/static/catalog-pg-statistic-ext.html) that holds data about the created statistics, but it gets updated only by an `ANALYZE` command. Each row in the view corresponds to a created statistics and includes the type and relationship:

```sql
SELECT * FROM pg_statistic_ext;

-[ RECORD 1 ]---|---------------------
stxrelid        | 51015
stxname         | stat_value_incoming
stxnamespace    | 2200
stxowner        | 16384
stxkeys         | 2 6
stxkind         | {f}
stxndistinct    | 
stxdependencies | {"2 => 6": 1.000000}
-[ RECORD 2 ]---|---------------------
stxrelid        | 51015
stxname         | stat_day_quarter
stxnamespace    | 2200
stxowner        | 16384
stxkeys         | 3 4
stxkind         | {f}
stxndistinct    | 
stxdependencies | {"3 => 4": 1.000000}
-[ RECORD 3 ]---|---------------------
stxrelid        | 51015
stxname         | stat_day_year
stxnamespace    | 2200
stxowner        | 16384
stxkeys         | 3 5
stxkind         | {f}
stxndistinct    | 
stxdependencies | {"3 => 5": 1.000000}
```

Therefore column `3` determines both columns `4` and `5` (see `stxkeys`) by a *functional* `{f}` dependency, and in particular target columns are fully computed by their dependendant column (100%) as reported in the `stxdependencies` column. The same applies for column `2` (`value`) that determines column `6` (`incoming`).

What is now the query plan?

```sql
 EXPLAIN 
SELECT *
FROM expenses
WHERE incoming = true
AND   value > 0.0::money
AND   day = '2016-7-19'::date
AND   year = 2016;
                                               QUERY PLAN                                               
--------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..35251.67 rows=1930 width=32)
   Workers Planned: 2
   ->  Parallel Seq Scan on expenses  (cost=0.00..34058.67 rows=804 width=32)
         Filter: (incoming AND (day = '2016-07-19'::date) AND (year = 2016) AND (value > (0.0)::money))
```

As you can see the planner now can estimates almost exactly the number of tuples that will be returned (*1930* exstimated against the *1982* actual). The fact is that now the planner knows the predicate pairs on `incoming` and `value`, as well as `day` and `year` are now tied due to a functional dependency and therefore it will not multiplicate each individual ratio to get the final filtering ratio.


## What about indexes?

Initially I thought that the new extended statistics could *automagically* understand also index depdencies, so that an index built on an expression of `day` can be used to retrieve also `year`. Clearly, [I was wrong](https://www.postgresql.org/message-id/CAKoxK%2B6C8CKdbYbbyNeYnc5aiDk%3DG-k-iDyDZMcmjJATqkLM9w%40mail.gmail.com), and that proves that *I have to do a better job learning this new feaure!*.
