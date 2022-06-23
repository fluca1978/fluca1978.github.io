---
layout: post
title:  "Ordinality in function queries"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A trick about queryies that involves function.

# Ordinality in fuction queries

The PostgreSQL `SELECT` statement allows you to query function that return *result set* (either a `SET OF` or `TABLE`), that are used as source of tuple for the query itself.
<br/>
There is nothing surprising about that!
<br/>
However, the `SELECT` statement, when invoked against a function that provides a result set, allows an extra clause to appear: `[WITH ORDINALITY](https://www.postgresql.org/docs/14/sql-select.html){:target="_blank"}`. This clause adds a column to the result set with a numerator (of type `bigint`) representing the number of the tuple as got from the function.
<br/>
<br/>
Why is this important? Because you don't need your function to provide by itself a kind of *tuple numerator*.


## `WITH ORDINALITY` in action

Let's take a simple example to understand how it works. Let's create a function that returns a table:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION animals( l int DEFAULT 5,
                                    animal text DEFAULT 'cat',
                                    owner text DEFAULT 'nobody' )
RETURNS TABLE( pk int, description text, mood text )
AS $CODE$
DECLARE
        i int := 0;
        j int := 0;
BEGIN
        FOR i IN 1 .. l LOOP
            pk          := i;
            description := format( '%s #%s owned by %s', animal, i, owner );
            j           := random() * 100;
            IF j % 2 = 0 THEN
               mood     := 'good';
            ELSE
              mood      := 'bad';
            END IF;

            RAISE DEBUG 'Generating % # % with mood %', animal, i, mood;
            RETURN NEXT;
        END LOOP;
RETURN;

END
$CODE$
LANGUAGE plpgsql;
```

<br/>
<br/>

The above function `animals()` produced an output with a simple name of the animal (numerated), the index of the generated tuple (i.e., a numerator) and a randomly select mood.
<br/>
It is clearly easy to test it out:

<br/>
<br/>

``` sql
testdb=> SELECT * FROM animals();
 pk |      description       | mood
----+------------------------+------
  1 | cat #1 owned by nobody | bad
  2 | cat #2 owned by nobody | good
  3 | cat #3 owned by nobody | bad
  4 | cat #4 owned by nobody | bad
  5 | cat #5 owned by nobody | bad
(5 rows)
```
<br/>
<br/>

The `pk` column contains the numerator of the generated tuple, so we know that the `cat #1` tuple has been generated first, the `cat #2` as second and so on.
<br/>
Let's kick `WITH ORDINALITY` in:


<br/>
<br/>

``` sql
testdb=> SELECT * FROM animals() WITH ORDINALITY;
 pk |      description       | mood | ordinality
----+------------------------+------+------------
  1 | cat #1 owned by nobody | good |          1
  2 | cat #2 owned by nobody | bad  |          2
  3 | cat #3 owned by nobody | good |          3
  4 | cat #4 owned by nobody | good |          4
  5 | cat #5 owned by nobody | good |          5

```
<br/>
<br/>

The `WITH ORDINALITY` clause must follow the function it will be apply onto. Such clause appends a new column to the result set, by default named `ordinality` with a progressive numerator. Note how `pk` and `ordinality` contain the very same value: **`WITH ORDINALITY` is keeping track for you of the tuple produced by the result set stream (the function)**, so you don't need to compute by yourself.
<br/>
Clearly, this works also with a reordering of the tuples, because the clause does not numerate the appearance of the tuples, rather the *instant* (or better, the *sequence*) a tuple has been added to the result set:

<br/>
<br/>

``` sql
testdb=> SELECT * FROM animals() WITH ORDINALITY
         ORDER BY random();
 pk |      description       | mood | ordinality
----+------------------------+------+------------
  4 | cat #4 owned by nobody | good |          4
  2 | cat #2 owned by nobody | good |          2
  3 | cat #3 owned by nobody | good |          3
  5 | cat #5 owned by nobody | good |          5
  1 | cat #1 owned by nobody | good |          1
(5 rows)
```
<br/>
<br/>


It is also possible to rename the `ordinality` column with an alias, like the following:


<br/>
<br/>

``` sql
testdb=> SELECT * FROM animals() WITH ORDINALITY
                       AS cat(i, name, mood, n)
                       ORDER BY random();
 i |          name          | mood | n
---+------------------------+------+---
 4 | cat #4 owned by nobody | good | 4
 1 | cat #1 owned by nobody | good | 1
 2 | cat #2 owned by nobody | bad  | 2
 5 | cat #5 owned by nobody | bad  | 5
 3 | cat #3 owned by nobody | good | 3
(5 rows)

```
<br/>
<br/>

*Clearly, you have to alias the whole result set, not a single column!*

## `WITH ORDINALITY` as a filtering condition

Having the automatically named `ordinality` column, or a custom chosen named column, it is possible to add such column to the `WHERE` clause of a query:

<br/>
<br/>

``` sql
testdb=> SELECT * FROM animals() WITH ORDINALITY                                                                                        AS cat(i, name, mood, n)
                       WHERE n % 2 = 0
                       ORDER BY random();
 i |          name          | mood | n
---+------------------------+------+---
 4 | cat #4 owned by nobody | bad  | 4
 2 | cat #2 owned by nobody | bad  | 2
(2 rows)

```
<br/>
<br/>

as you can see, the above query filters on the `n` column to get only even tuples.


## `WITH ORDINALITY` vs `row_number()`

You may think that the window function `[row_number()](https://www.postgresql.org/docs/14/functions-window.html){:target="_blank"}` does the same job as `WITH ORDINALITY`, at least in the function call scenario.
However, the `row_number()` window function is a different beast, and can work on a window defined against the result set ordinality. *In short, window functions cover a diferent set of problems!*
<br/>
Therefore, even if the following seems to produce the very same result:

<br/>
<br/>
```sql
testdb=> SELECT *, row_number() OVER () FROM animals() WITH ORDINALITY;
 pk |      description       | mood | ordinality | row_number
----+------------------------+------+------------+------------
  1 | cat #1 owned by nobody | good |          1 |          1
  2 | cat #2 owned by nobody | bad  |          2 |          2
  3 | cat #3 owned by nobody | bad  |          3 |          3
  4 | cat #4 owned by nobody | good |          4 |          4
  5 | cat #5 owned by nobody | bad  |          5 |          5
(5 rows)

```
<br/>
<br/>

as soon as you define your partition to number in a more specialized way you see different results:

<br/>
<br/>

``` sql
testdb=> SELECT *, row_number() OVER ( order by pk desc ) FROM animals() WITH ORDINALITY;
 pk |      description       | mood | ordinality | row_number
----+------------------------+------+------------+------------
  5 | cat #5 owned by nobody | bad  |          5 |          1
  4 | cat #4 owned by nobody | bad  |          4 |          2
  3 | cat #3 owned by nobody | bad  |          3 |          3
  2 | cat #2 owned by nobody | good |          2 |          4
  1 | cat #1 owned by nobody | good |          1 |          5

```
<br/>
<br/>

In the above, you can see that the last row produced by the function (this with `ordinality` set to `5`) is the first row encountered by `row_number()`.
<br/>
Another example of different results can be quickly obtained when joining:

<br/>
<br/>

``` sql
testdb=> SELECT *, row_number() OVER ()
         FROM animals() WITH ORDINALITY,
         generate_series(1, 3) WITH ORDINALITY as x(gs, counter);
 pk |      description       | mood | ordinality | gs | counter | row_number
----+------------------------+------+------------+----+---------+------------
  1 | cat #1 owned by nobody | bad  |          1 |  1 |       1 |          1
  2 | cat #2 owned by nobody | good |          2 |  1 |       1 |          2
  3 | cat #3 owned by nobody | good |          3 |  1 |       1 |          3
  4 | cat #4 owned by nobody | good |          4 |  1 |       1 |          4
  5 | cat #5 owned by nobody | good |          5 |  1 |       1 |          5
  1 | cat #1 owned by nobody | bad  |          1 |  2 |       2 |          6
  2 | cat #2 owned by nobody | good |          2 |  2 |       2 |          7
  3 | cat #3 owned by nobody | good |          3 |  2 |       2 |          8
  4 | cat #4 owned by nobody | good |          4 |  2 |       2 |          9
  5 | cat #5 owned by nobody | good |          5 |  2 |       2 |         10
  1 | cat #1 owned by nobody | bad  |          1 |  3 |       3 |         11
  2 | cat #2 owned by nobody | good |          2 |  3 |       3 |         12
  3 | cat #3 owned by nobody | good |          3 |  3 |       3 |         13
  4 | cat #4 owned by nobody | good |          4 |  3 |       3 |         14
  5 | cat #5 owned by nobody | good |          5 |  3 |       3 |         15
(15 rows)

```

<br/>
<br/>

For every `generate_series()` tuple (column `counter`) there are five `animals()` tuples (column `ordinality`), each one progressively tracked by `row_number()`.


## Conclusions

Why is this ordinality thing important?
<br/>
It may happen that you are tempted to include into your function result sets some extra information that will ease the post-processing of the result set itself. This practice should be avoided when the "external world" (i.e., the query using the function) is able to add such extra information by itself. You will not waste resources, but also keep your code cleaner and more readable.
