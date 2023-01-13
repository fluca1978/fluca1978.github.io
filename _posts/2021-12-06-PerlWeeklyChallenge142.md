---
layout: post
title:  "Perl Weekly Challenge 142: divide and sleep!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 142: divide and sleep!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 142](https://perlweeklychallenge.org/blog/perl-weekly-challenge-142/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg)





<a name="task1"></a>
## PWC 142 - Task 1

The first task was quite easy to solve: given two numbers, find out all the divisors of the former one that end with the second number. One row to rule the rask:

<br/>
<br/>
```raku
sub MAIN( Int $m where { $m > 1 }
          , Int $n where { $n > 0 && $m > $n } ) {
   ( 1 .. $m ).grep( $m %% * ).grep( * ~~ / ^ \d* $n $ / ).elems.say;
}
 
 ```
<br/>
<br/>

The idea is simple: I produce all the numbers from `1` to `$m`, the number I must seek the divisors of. Then I do `grep` all the numbers keeping only those that are "real" divisors, and then I do `grep` again searching for a regular expression that makes the given divisor ending with the `$n` digit. 
At the end, I count the found elements and output the value.



<a name="task2"></a>
## PWC 142 - Task 2

The second task was about implementing the *sleep sort*, a simple and resource-consuming sort mechanism that requires the creation of one thread for every value to sort. Each thread has to sleep for the specified time according to the value it has to sort, and then the thread emits the value. In this way, apart from the time required to create the threads, each value will be emitted sorted.
<br/>
`Promise`s are the way to go in Raku:

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.grep( * ~~ Int ).elems == @n.elems  } ) {
    my @threads;
    for @n -> $sleep {
        @threads.push: Promise.in( $sleep.Int ).then( { $sleep.say } );
    }

    await @threads;
}
 
```
<br/>
<br/>

I do create a list of `@threads`, each one attached to a `Promise` that will be kept `in` the sepcified amount of seconds. Once the promise is kept, the value "attached" to the promise will be printed on standard output. And then the program waits for all the promises to finish.
<br/>
It is important to note that the variable `$sleep` as to be made explicit, because `$_` within the `then` block of the `Promise` will result in the promise itself.


<a name="task1pg"></a>
## PWC 142 - Task 1 in PostgreSQL `plpgsql`

A *recursive CTE* could solve the problem:
<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
  f_divisors_last_digit( m int, n int )
  RETURNS SETOF int
AS $CODE$
  WITH RECURSIVE numbers AS (
    SELECT 1 as num
     UNION
    SELECT num + 1
      FROM numbers
     WHERE num + 1 <= m )
  , divisors AS (
    SELECT num as div
      FROM numbers
      WHERE m % num = 0 )
  SELECT count(*)
  FROM divisors
  WHERE div::text LIKE ( '%' || n::text );

  $CODE$
  LANGUAGE sql;
 
```
<br/>
<br/>

The `numbers` part generates the sequence of all the numbers from `1` to `m`, and corresponds to the `( 1 .. $m)` part in the Raku solution. The `divisors` part excludes from `numbers` those that are not divisors of `m`, and correspond to the `grep( $m %% * )` part in Raku.
Last, the final `SELECT` counts all the divisors that end with `n` using `LIKE`, that is the SQL way to search for a pattern matching.

<a name="task2pg"></a>
## PWC 142 - Task 2 in PostgreSQL `plpgsql**

**There is no way to implement threading in PostgreSQL**, and this is because the database must decide how many threads (or better, processes) to launch. Remember: *SQL is a declarative language* .
<br/>
However, there is a way to simulate the same behavior by means of opening a connection for every number to sort, make the connection to sleep and then print out the value.
<br/>
This is quite easy to achieve with `psql` capabilities:

<br/>
<br/>

``` shell
 #!/bin/sh

for i in $*; do
    psql  -At -c "SELECT pg_sleep( $i ); SELECT $i;"   &
done

```
<br/>
<br/>

The `-At` instruments `psql` to start the unaligned mode and print out only the results, without any header. The `-c` tells `psql` to execute immediatly the following SQL statement, that is made by two parts:
- `pg_sleep()` is a function that tells the connection to sleep for the specified amount of seconds;
- `SELECT` prints out the specified number.

<br/>
Last, every `psql` instance is pushed to background, so to continue with the following number to sort.
