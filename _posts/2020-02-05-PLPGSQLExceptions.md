---
layout: post
title:  "PL/PgSQL Exception and XIDs"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A few considerations on how exceptions are handled in PL/PgSQL.

# PL/PgSQL Exception and XIDs

I read the blog post [The strange case of the EXCEPTION block](https://pgdba.org/post/2020/02/exception_block/){:target="_blank"} where the author was claiming that an `EXCEPTION` block in a PL/PgSQL function was incrementing the transaction id (*xid*).
<br/>
Somehow, this was not very surprising to me.
<br/>
Why? That reminded me immediatly [my own question on the general mailing list](https://www.postgresql.org/message-id/CAKoxK%2B5Wm6RazPnU8AqB97XRqx5zbY7us00QSVrQgobbgmf8hQ%40mail.gmail.com){:target="_blank"} when I was observing a very similar behaviour within `psql`. In particular, [this answer was illuminating](https://www.postgresql.org/message-id/20171106125330.gcozh4ijjrkn6shq%40alvherre.pgsql){:target="_blank"}:

       something is using subtransactions there.  
       My first guess would be that
       there are triggers with EXCEPTION blocks

## My Guess About How Exceptions Are Handled

**I think PL/PgSQL is using subtransactions (or savepoints) to handle exceptions**. 
<br/>
Why?
<br/>
Well, if you think about when you *catch* and exception you probably want to resume your execution, that is you must have a way to *rollback* your unit of work and start over again.


## See Transactions in Action!

It is possible to inspect the transactions in action with a simple function and a table to abuse.
<br/>
There is no need to play around with `VACUUM FREEZE` and `age()` as the original author says.
<br/>
Let's see the function:

```sql
CREATE OR REPLACE FUNCTION f_loop( b int DEFAULT 0, e int DEFAULT 10 )
  RETURNS VOID
  AS $$
  BEGIN
      RAISE DEBUG 'TXID of the function (here should not be assigned) function: % %',
                      txid_current_if_assigned(),
                      txid_status( txid_current_if_assigned() );
    FOR f IN b .. e LOOP
      BEGIN

        RAISE DEBUG 'Before INSERT of % TXID: %  SNAPSHOT: %',
                    f,
                    txid_current_if_assigned(),
                    txid_current_snapshot();

       INSERT INTO foo( i ) VALUES( f );


      RAISE DEBUG 'After INSERT of % TXID: %  SNAPSHOT: %',
                      f,
                      txid_current_if_assigned(),
                      txid_current_snapshot();

       EXCEPTION
        WHEN UNIQUE_VIOLATION
           THEN   RAISE DEBUG 'Exception for % TXID: %  SNAPSHOT: %',
                        f,
                        txid_current_if_assigned(),
                        txid_current_snapshot();

       END;
    END LOOP;
  END;
  $$
  LANGUAGE plpgsql;
```

The function accepts a begin and end indexes and loop thru every value between them, trying to insert the value into a table. At every step, including the exception, we inspect `txid_current_if_assigned()`, that reports the transaction ID (`xid`) and `txid_current_snapshot()` that provides the current snapshot, that means roughly the minimum and maximum xid this transaction is "flying" over.

<br/>
The definition of the table is pretty straightforward: it has a single column with a `UNIQUE` constraint on it. That's the constraint the function is going to violate.

```sql
CREATE TABLE foo ( i int PRIMARY KEY );
```

### First Run: No Exceptions

Since the table is empty, inserting values from `1` to `10` does not produce any exception.

```sql
testdb=> SELECT f_loop( 1, 10 );  
DEBUG:  TXID of the function (here should not be assigned) function: <NULL> <NULL>
DEBUG:  Before INSERT of 1 TXID: <NULL>  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 1 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 2 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 2 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 3 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 3 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 4 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 4 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 5 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 5 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 6 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 6 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 7 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 7 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 8 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 8 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 9 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 9 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  Before INSERT of 10 TXID: 4748  SNAPSHOT: 4748:4748:
DEBUG:  After INSERT of 10 TXID: 4748  SNAPSHOT: 4748:4748:
 f_loop 
--------
 
(1 row)
```

In the very first run the `xid` is `NULL` because the function has not (yet) modified anything. That's why I use `txid_current_if_assigned()` instead of `txid_current()` to avoid wasting a number. Once the function starts modifying the data (i.e., after the very first `INSERT`) the transaction is *promoted* from virtual to *concrete* and so a `xid` is assigned.
<br/>
Since no exception at all is raised, the `xid` of the function is *fixed* and so is the snapshot.

## Second Run: Half of Exceptions

Let's run it with some numbers overlapping, so that half of the values are inserted succesfully and half throw an exception.

```sql
testdb=> SELECT f_loop( 5, 15 ); 
DEBUG:  TXID of the function (here should not be assigned) function: <NULL> <NULL>
DEBUG:  Before INSERT of 5 TXID: <NULL>  SNAPSHOT: 4760:4760:
DEBUG:  Exception for 5 TXID: 4760  SNAPSHOT: 4760:4762:
DEBUG:  Before INSERT of 6 TXID: 4760  SNAPSHOT: 4760:4762:
DEBUG:  Exception for 6 TXID: 4760  SNAPSHOT: 4760:4763:
DEBUG:  Before INSERT of 7 TXID: 4760  SNAPSHOT: 4760:4763:
DEBUG:  Exception for 7 TXID: 4760  SNAPSHOT: 4760:4764:
DEBUG:  Before INSERT of 8 TXID: 4760  SNAPSHOT: 4760:4764:
DEBUG:  Exception for 8 TXID: 4760  SNAPSHOT: 4760:4765:
DEBUG:  Before INSERT of 9 TXID: 4760  SNAPSHOT: 4760:4765:
DEBUG:  Exception for 9 TXID: 4760  SNAPSHOT: 4760:4766:
DEBUG:  Before INSERT of 10 TXID: 4760  SNAPSHOT: 4760:4766:
DEBUG:  Exception for 10 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  Before INSERT of 11 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  After INSERT of 11 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  Before INSERT of 12 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  After INSERT of 12 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  Before INSERT of 13 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  After INSERT of 13 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  Before INSERT of 14 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  After INSERT of 14 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  Before INSERT of 15 TXID: 4760  SNAPSHOT: 4760:4767:
DEBUG:  After INSERT of 15 TXID: 4760  SNAPSHOT: 4760:4767:
 f_loop 
--------
 
(1 row)
```

As you can see, in the first five numbers there's an exception reported. The `xid` of the function remains the same, but the snapshot grows by 6 transactions identifiers (one for the function, five for the subtransactions).
After that, the remaining five values are succesfully inserted and so the snapshot does not grow anymore.

### Where are these Subtransactions?

If you now inspect the MVCC values for the table, you can see that every value inserted has a different transaction id `xmin`, without any regard to the fact that the function call did catch an exception or not.

```sql
testdb=> SELECT xmin,xmax, cmin, cmax, * FROM foo;
 xmin | xmax | cmin | cmax | i  
------|------|------|------|----
 4749 |    0 |    0 |    0 |  1
 4750 |    0 |    1 |    1 |  2
 4751 |    0 |    2 |    2 |  3
 4752 |    0 |    3 |    3 |  4
 4753 |    0 |    4 |    4 |  5
 4754 |    0 |    5 |    5 |  6
 4755 |    0 |    6 |    6 |  7
 4756 |    0 |    7 |    7 |  8
 4757 |    0 |    8 |    8 |  9
 4758 |    0 |    9 |    9 | 10
 4767 |    0 |    6 |    6 | 11
 4768 |    0 |    7 |    7 | 12
 4769 |    0 |    8 |    8 | 13
 4770 |    0 |    9 |    9 | 14
 4771 |    0 |   10 |   10 | 15
(15 rows)
``**

### How to Simulate the Same Behavior

**Savepoints** do pretty much the same! Therefore, let's truncate the table and insert new values in it with an explicit transaction and savepoints:



```sql
testdb=> TRUNCATE foo;
TRUNCATE TABLE
testdb=> BEGIN;
BEGIN
testdb=> 
testdb=> INSERT INTO foo( i ) VALUES( 1 );
INSERT 0 1
testdb=> SAVEPOINT S1;
SAVEPOINT
testdb=> 
testdb=> INSERT INTO foo( i ) VALUES( 2 );
INSERT 0 1
testdb=> SAVEPOINT S2;
SAVEPOINT
testdb=> 
testdb=> INSERT INTO foo( i ) VALUES( 3 );
INSERT 0 1
testdb=> SAVEPOINT S3;
SAVEPOINT
testdb=> 
testdb=> COMMIT;
COMMIT
testdb=> SELECT xmin,xmax, cmin, cmax, * FROM foo;
 xmin | xmax | cmin | cmax | i 
------|------|------|------|---
 4779 |    0 |    0 |    0 | 1
 4780 |    0 |    1 |    1 | 2
 4781 |    0 |    2 |    2 | 3
(3 rows)
```

As you can see the `xmin` is incremented continuosly by every `INSERT`.


# Conclusions

Exception are quite clearly implemented in PL/PgSQL (and possibly in other languages) by means of subtransactions. At least, the behavior is pretty much reproducible.
