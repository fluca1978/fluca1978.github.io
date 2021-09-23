---
layout: post
title:  "Restarting a sequence: how hard could it be? (PostgreSQL and Oracle)" 
author: Luca Ferrari
tags:
- postgresql
- oracle
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How hard could it be to reset a sequence?

# Restarting a sequence: how hard could it be? (PostgreSQL and Oracle)

One reason I like PostgreSQL so much is that it makes me feel at home: it has a very consistent and coherent interface to its objects.
An example of this, is the management of sequences: `ALTER SEQUENCE` allows you to modify pretty much every detail about a sequence, in particular to *restart* it from its initial value.
<br/>
Let's see this in action:

<br/>
<br/>
```sql
testdb=> create sequence batch_seq 
         increment by 1 start with 1;
CREATE SEQUENCE

testdb=> do $$
declare
  i int;
begin
  for i in 1..100 loop
     perform nextval( 'batch_seq' );
  end loop;
end
$$
;
DO


testdb=> select currval( 'batch_seq' );
 currval 
---------
     100

```
<br/>
<br/>

In the above piece of code, I've created a `batch_seq` and queried it one hundred times, so that the current value of the sequence is holding `100`.
<br/>
<br/>
How is it possible to make the sequence start over again?
<br/>
A first possibility is to use the `setval` function:

<br/>
<br/>
```sql
testdb=> select setval( 'batch_seq', 1 );
 setval 
--------
      1


testdb=> select currval( 'batch_seq' );
 currval 
---------
       1

```
<br/>
<br/>



Another option is to use `ALTER SEQUENCE`, that is a command aimed to this purpose (and others):


<br/>
<br/>
```sql
testdb=> alter sequence batch_seq restart;
ALTER SEQUENCE

testdb=> select nextval( 'batch_seq' );   
 nextval 
---------
       1

```
<br/>
<br/>
An important thing to note here, is that the only option specified has been `RESTART`, that is the sequence already knows what *restarting* means: it means *reset to its original starting value*.
<br/>
It is also possible to specify a specific value for the restarting:


<br/>
<br/>
```sql
testdb=> alter sequence batch_seq restart with 666;
ALTER SEQUENCE
        
testdb=> select nextval( 'batch_seq' );
 nextval 
---------
     666

```
<br/>
<br/>
*That's so simple!* 
<br/>
The above behaviour is guaranteed back to the `8.1` PostgreSQL version (and probably even before): [see the old documentation here](https://www.postgresql.org/docs/8.1/sql-altersequence.html){:target="_blank"}.



## Wait, what about `currval()`?

The careful reader has probably noted that I used `nextval()` to see if the reset of a sequence worked, instead of `currval()`. The reason can be found in the [official documentation](https://www.postgresql.org/docs/13/functions-sequence.html){:target="_blank"}: *Returns the value most recently obtained by nextval for this sequence ***in the current session** . *
<br/>
It is easy to test this:

<br/>
<br/>
```sql
testdb=> select nextval( 'batch_seq' );
 nextval 
---------
     667


testdb=> alter sequence batch_seq restart with 999;
ALTER SEQUENCE

testdb=> select currval( 'batch_seq' );
 currval 
---------
     667


testdb=> select nextval( 'batch_seq' );
 nextval 
---------
     999

```
<br/>
<br/>

As you can see, after an `ALTER SEQUENCE RESTART` the `currval()` result remains unchanged (it is the last polled value within the current session), while `nextval()` (that actually queries the sequence) provides the right and expected value.



# What about Oracle sequences?

Oracle provides a powerful `ALTER SEQUENCE` command only in recent versions. For older versions, the [official documentation for the command `ALTER SEQUENCE`](){:target="_blank"} clearly states that **To restart the sequence at a different number, you must drop and re-create it**!

<br/>
<br/>
Err... what?
<br/>
<br/>

Until version 18: `ALTER SEQUENCE` cannot *restart* the sequence. What is then the solution? You need to trigger a sequence update:
- change the increment of the sequence to effectively subtract values;
- ask the sequence a new value, so that it applies the subtraction;
- set the increment to its correct value.

<br/>
This means you have to do something like the following:


<br/>
<br/>
```sql
SQL> select batch_seq.nextval from dual; 
SQL> alter sequence batch_seq  increment by -666;
SQL> select batch_seq.nextval from dual; 
SQL> alter sequence batch_seq  increment by 1;
```
<br/>
<br/>

I don't like this approach very much, because it is error prone and requires you to do some computation ensuring you are not going to go outside the sequence boundaries.

<br/>
<br/>
In recent versions of Oracle Database (e.g., `21`), the [`ALTER SEQUENCE` command works as in PostgreSQL, i.e., as in the standard SQL](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/ALTER-SEQUENCE.html#GUID-A6468B63-E7C9-4EF0-B048-82FE2449B26D){:target="_blank"}, and this is good, of course.
<br/>
With a quick search for within the [Oracle documentation about `ALTER SEQUENCE`](https://docs.oracle.com/search/?q=alter+sequence&category=database&product=en%2Fdatabase%2Foracle%2Foracle-database){:target="_blank"}, the *right* behaviour has been introduced in Oracle `18` and next. Therefore, if you are facing a previous Oracle version, you need to do the above set of commands to manually adjust the sequences.




# Conclusions

PostgreSQL has a very strict approach to the SQL standard, that roots even in old versions. Unluckily, Oracle is not the same, and older versions require some tricks to simulate the PostgreSQL behavior.
<br/>
This is not meant to be a flame or a comparison, it simply indicates how counter-intuitive could be to handle Oracle once you have been used to PostgreSQL!
