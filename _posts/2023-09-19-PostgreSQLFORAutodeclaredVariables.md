---
layout: post
title:  "FOR loops automatically declared variables in PL/PgSQL"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
In PL/PgSQL FOR loops the iterator is automatically declared, and this could bring some problems.

# FOR loops automatically declared variables in PL/PgSQL

Consider the following simple function that returns a table made by three columns:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
public.a_table()
RETURNS TABLE( i int, j int, k int )
AS $CODE$

BEGIN
	FOR i IN 1 .. 2 LOOP
	    FOR j IN 1 .. 2 LOOP
	    	FOR k IN 1 .. 2 LOOP
		    RAISE INFO 'i=%, j=%, k=%', i, j, k;
		    RETURN NEXT;
		END LOOP;
	    END LOOP;
	END LOOP;
END

$CODE$
LANGUAGE plpgsql
VOLATILE
;
```
<br/>
<br/>

What is the result of invoking the above function?
<br/>
Depending on how you know the `FOR` loop in PL/PgSQL, it could be surprising:

<br/>
<br/>
```sql
testdb=> select * from a_table();
INFO:  i=1, j=1, k=1
INFO:  i=1, j=1, k=2
INFO:  i=1, j=2, k=1
INFO:  i=1, j=2, k=2
INFO:  i=2, j=1, k=1
INFO:  i=2, j=1, k=2
INFO:  i=2, j=2, k=1
INFO:  i=2, j=2, k=2
 i | j | k
---+---+---
   |   |
   |   |
   |   |
   |   |
   |   |
   |   |
   |   |
   |   |
(8 rows)

```
<br/>
<br/>


Why is the result set empty even if the variables have values?
<br/>
Because the `FOR` iterator is automatically declared and scoped to the loop itself. The [PostgreSQL Documentation](https://www.postgresql.org/docs/16/plpgsql-control-structures.html){:target="_blank"} explains it:

>  The variable name is automatically defined as type integer and exists only inside the loop (any existing definition of the variable name is ignored


It should be clear that I'm referring to the *integer `FOR` loop variant* here. However, the problem is that while `i`, `j` and `k` are defined as variables for the function (the returning columns), the `FOR` loops create variables with the same name but an innser scope, so that it is not possible to refer to the returning columns.

<br/>

Please note that the problem is not caught even with [the warnings about shadowed variables](https://www.postgresql.org/docs/16/plpgsql-development-tips.html){:target="_blank"} :

<br/>
<br/>
```sql
testdb=> SET plpgsql.extra_warnings TO 'shadowed_variables';
SET
testdb=> select * from a_table();
INFO:  i=1, j=1, k=1
INFO:  i=1, j=1, k=2
INFO:  i=1, j=2, k=1
INFO:  i=1, j=2, k=2
INFO:  i=2, j=1, k=1
INFO:  i=2, j=1, k=2
INFO:  i=2, j=2, k=1
INFO:  i=2, j=2, k=2
 i | j | k
---+---+---
   |   |
   |   |
   |   |
   |   |
   |   |
   |   |
   |   |
   |   |
(8 rows)

```
<br/>
<br/>



Therefore, so far, the only solution is to choose appropriately the names of iterators, and of course to set the returnig variables accordingly:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
public.a_table()
RETURNS TABLE( i int, j int, k int )
AS $CODE$

BEGIN
	FOR ii IN 1 .. 2 LOOP
	    FOR jj IN 1 .. 2 LOOP
	    	FOR kk IN 1 .. 2 LOOP
		    i := ii;
		    j := jj;
		    k := kk;
		    RAISE INFO 'i=%, j=%, k=%', i, j, k;
		    RETURN NEXT;
		END LOOP;
	    END LOOP;
	END LOOP;
END

$CODE$
LANGUAGE plpgsql
VOLATILE
;

```
<br/>
<br/>


The above in fact results in what you probably are expecting:

<br/>
<br/>
```sql
testdb=> select * from a_table();
INFO:  i=1, j=1, k=1
INFO:  i=1, j=1, k=2
INFO:  i=1, j=2, k=1
INFO:  i=1, j=2, k=2
INFO:  i=2, j=1, k=1
INFO:  i=2, j=1, k=2
INFO:  i=2, j=2, k=1
INFO:  i=2, j=2, k=2
 i | j | k
---+---+---
 1 | 1 | 1
 1 | 1 | 2
 1 | 2 | 1
 1 | 2 | 2
 2 | 1 | 1
 2 | 1 | 2
 2 | 2 | 1
 2 | 2 | 2

```
<br/>
<br/>




## Using `plpgsql_check` as a possible help

*This is a post update thanks to the comment of **Pavel Stěhule**  on 2023-09-20*.

The [plpgsql_check](https://github.com/okbob/plpgsql_check){:target="_blank"} extension could help in finding out the above described problem.
Covering `plpgsql_check` here is out of the scope, however this is how the extension can provide some help:

<br/>
<br/>
```sql
testdb=# CREATE EXTENSION plpgsql_check;
CRATE EXTENSION

testdb=# SELECT message, level FROM plpgsql_check_function_tb( 'a_table()' );
           message           |     level
-----------------------------+---------------
 unmodified OUT variable "i" | warning extra
 unmodified OUT variable "j" | warning extra
 unmodified OUT variable "k" | warning extra
(3 rows)


```
<br/>
<br/>


As you can see, the check does not *understand* the effective problem, that is that the variables are all masked out by the context defined in the `FOR` loops, but at least it reveals that the output variables have not been modified along the function code. Knowing that such variables have not been modified means that what the function is *expecting* to achieve is probably not, and that will trigger some extra check by the developers.
