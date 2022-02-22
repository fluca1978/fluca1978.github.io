---
layout: post
title:  "Perl Weekly Challenge 153: recursive CTEs"
author: Luca Ferrari
tags:
- perlweeklychallenge
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 153: recursive CTEs

This is a short tour about my solutions to the  [Challenge 153](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0153/){:target="_blank"} done in PostgreSQL.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>


<a name="task1"></a>
## PWC 153 - Task 1

This task was about producing *left factorial numbers*, where each value is computed by summing all the previously computed factorials.
<br/>
I decided to implement it on top of a *recursive CTE* named `factorials` that, well, computes factorials. That was the easy part, then I needed to compute the `sum` of all the values less than the current one. Let's use a `LATERAL JOIN` for the task:
<br/>


<br/>
<br/>
```sql
with recursive factorials as
(
   SELECT 0::numeric as num
         ,1::numeric as fac

   UNION

   SELECT f.num + 1
         , ( f.num + 1 ) * f.fac
   FROM factorials f
   WHERE f.num < 1000
)
SELECT f.num, sum( w.fac ) as left_factorial
FROM factorials f, LATERAL
( SELECT ff.fac FROM factorials ff WHERE ff.num < f.num ORDER BY ff.num ) w
WHERE f.num <= 10
GROUP BY f.num, f.fac
ORDER BY f.num
;

 ```
<br/>
<br/>


I limit the number of work to be done to `10`, as requested by the task. The `factorials` CTE computes all the factorials, and then I join `LATERAL` with a subquery `w` that selects all the factorial values for entries less then current one. Therefore, using the built-in `sum` function in the outer query solves the problem.
<br/>
Clearly, this is not a particularly efficient solution, but it is a good example of what recursive CTEs and `LATERAL` an do when combined together.


<a name="task2"></a>
## PWC 153 - Task 2

Similar to the previous task, but simpler: see if a given number is made by digits that, when summed as factorials, provide the number itself. As an example, `145` is a number that can be expressed as `!1 + 4! + 5!`.
<br/>
Having a recursive CTE to compute factorials from the previous task, I decided to use the same starting point. However, this time, I used a `psql` variable named `needle` to which I assign the value I want to test:

<br/>
<br/>
```sql
\set needle 145

with recursive factorials as
(
   SELECT 0::numeric as num
         ,1::numeric as fac

   UNION

   SELECT f.num + 1
         , ( f.num + 1 ) * f.fac
   FROM factorials f
   WHERE f.num < 1000
)
SELECT CASE sum( f.fac ) WHEN :needle THEN :needle || ' OK' ELSE :needle || ' KO' END AS factorions
FROM factorials f JOIN regexp_split_to_table( :needle::text, '' ) w(n)
ON w.n = f.num::text
;
;

```
<br/>
<br/>

The trick here is that I join `factorials` with `regexp_split_to_table` that returns all the digits as a set of tuples. Then, in the outer query, I do `sum` the factorials of every digit and see if the result is still the `needle`, producing an `OK` string or a `KO` one.
