---
layout: post
title:  "Find transaction that are locked (and who to blame for!)"
author: Luca Ferrari
tags:
- PostgreSQL
permalink: /:year/:month/:day/:title.html
---
Have you ever wonder about what transaction (and user) is blocking another transaction? In other words, who to blame for keeping the database locked in any part? Here's a short and dirty way to dig a little into the problem.

# Find transaction that are locked (and who to blame for!)

PostgreSQL is great at looking in the sense it does its best effort to not lock at all! Nevertheless, in order to keep things coherent, even PostgreSQL requires locks to be acquired at diffent times and with different levels.

PostgreSQL provides a rich set of *statistical views* to introspect a running database, including single transaction activity and locks. Therefore, combining these information togethere there's a chance to find out **if and who is blocking due to not performing a lock release**.

The idea proposed in this article is quite simple and represents a **poor man way to find out locks information**. The example presented here is a live example I produced while preparing some teaching material.

The path to discover what and who is blocking is done thru three main steps:
1. **find out if there are blocked transactions**, this is done querying the activity statistic view having a glance at its state and/or the time it is running since (long running transactions could be possibly waiting for a lock to acquire);
2. **find out what blocked transaction are waiting for** and on the other hand, **what lock is not acquired**, and this is extracted by the lock statistical view, so that it is possible to see the lock not acquired and, by mean, the transaction that have acquired it;
3. **decide what to do** this is not an easy task. When the blocking transaction has been found, is it worth terminating or should it be kept running for another period of time in the case it volountary releases the lock? The choice strongly depends on the execution context of the database and transaction itself.

Please note that this post starts from an unknown situation, that is **no session or queries are shown here** because it is the worth and most true situation: an administrator should be able to understand what is going on without any clue except those provided by the database itself.
Nevertheless, investigating the situation will quickly reveals the queries causing the contention.

## Step 1: is anyone waiting for a lock acquisition?
Use the `pg_stat_activity` view to see if is there any transaction waiting for some kind of lock:

```sql
# SELECT query, backend_start, xact_start, query_start,
         state_change, state,
         now()::time - state_change::time AS locked_since,
         pid, wait_event_type, wait_event
  FROM pg_stat_activity
  WHERE wait_event_type IS NOT NULL
ORDER BY locked_since DESC;
```

The above query provides a list of *longest waiting* transactions, so for example:


```sql
-[ RECORD 1 ]---|-----------------------------------------
query           | UPDATE persona SET nome = upper( nome );
backend_start   | 2017-11-14 22:37:55.88444+01
xact_start      | 2017-11-14 22:37:58.186149+01
query_start     | 2017-11-14 22:38:16.362419+01
state_change    | 2017-11-14 22:38:16.362424+01
state           | active
locked_since    | 00:05:10.078716
pid             | 786
wait_event_type | Lock
wait_event      | transactionid
```

Now, the transaction associated to the process id `786` is waiting for a lock hold by another transaction (see `Lock` and `transactionid` in the `wait_event` columns). Moreover, the transaction has waited for around five minutes, and this is a quite clear indication that there is a possible *lock contention*.

It is therefore worth investigating some more.

Let's get some information out from the `pg_locks` view with the process identifier, or better, let's join `pg_locks` and `pg_stat_activity` so that it is possible to get information at glance:

```sql
# SELECT a.usename, a.application_name, a.datname, a.query,
         l.granted, l.mode, transactionid
    FROM pg_locks l
    JOIN pg_stat_activity a ON a.pid = l.pid
    WHERE granted = false AND a.pid = 786;
```

thanks to the process id it is possible to confirm that the transaction is waiting for a lock:

```sql
-[ RECORD 1 ]----|-----------------------------------------
usename          | luca
application_name | psql
datname          | testdb
query            | UPDATE persona SET nome = upper( nome );
granted          | f
mode             | ShareLock
transactionid    | 3031
```

Now, the transaction `3031` is the one that is preventing the `ShareLock` to be acquired.

## Step 2: see who is blocking

Having find out the transaction id that is preventing another one to acquire a lock it is possible to get some information about what such transaction is actually performing:

```sql
# SELECT a.usename, a.application_name, a.datname, a.query,
        l.granted, l.mode, transactionid,
        now()::time - a.state_change::time AS acquired_since,
        a.pid
   FROM pg_locks l
   JOIN pg_stat_activity a ON a.pid = l.pid
   WHERE granted = true AND transactionid = 3031;
```

Of course, we it is interesting to see acquired locks by transaction `3031`:

```sql
usename          | luca
application_name | psql
datname          | testdb
query            | UPDATE persona SET cognome = lower( cognome );
granted          | t
mode             | ExclusiveLock
transactionid    | 3031
acquired_since   | 00:27:20.264908
pid              | 780
```

Now, the transaction kept the `ExclusiveLock` for more than 27 minutes (this is ad-hoc example)!
It is quite clear something nasty is happening, but at least it is clear so far the transaction that is blocking other transactions and what query it is executing causing the lock to not be released.

## Step 3: what now?

Having found out the transaction not releasing the lock, the query it is performed, chances are an administrator can understand if this is a *normal behavior* or not.

Please note that the system provided us with the information about both running queries:
1. `UPDATE persona SET cognome = lower( cognome );`
2. `UPDATE persona SET nome = upper( nome );`

both queries perform a full update on the same table without any possible row lock acquisition (no filter specified), and this is a clear situation of contention. The scenario appear clear now.

In the case of emergency, this should help you to unlock waiting transactions:

```sql
# SELECT pg_terminate_backend( 780 );
```

**but really try to better understand what is going on before brutaly nuke a running transaction!**
