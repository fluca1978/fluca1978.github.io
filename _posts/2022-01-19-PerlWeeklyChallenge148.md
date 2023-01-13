---
layout: post
title:  "Perl Weekly Challenge 148: eban vs cardano"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 148: eban vs cardano

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 148](https://perlweeklychallenge.org/blog/perl-weekly-challenge-148/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg) and a [CTE only solution](#task2pgb)





<a name="task1"></a>
## PWC 148 - Task 1

This task was about finding out *Eban numbers*, those numbers that do not contain the letter `e` within their english name.
I was not really inspired by this task, so I brutally created a set of arrays that contains the numbers that do not copntain an `e` in their english name, and I `grep` for those numbers:


<br/>
<br/>
```raku
sub MAIN() {

    my @eban-units = 2, 4, 6;
    my @eban-teens = 12;
    my @eban-tens  = 3, 4, 5, 6;

    $_.say if @eban-units.grep( $_ ) for 1 .. 10;
    $_.say if @eban-teens.grep( $_ ) for 11 .. 19;
    $_.say if @eban-tens.grep( ( $_ / 10 ).Int ) && @eban-units.grep( $_ % 10 ) for 20 .. 100;

}


 ```
<br/>
<br/>

<a name="task2"></a>
## PWC 148 - Task 2

This has been a more difficult task, that required me to learn about the `=~=` *almost equality operator*.
The task required to print out the first five *Cardano numbers*, those that solve a particular equation given three variables.
First of all, I implemented a couple of functions that, given the three variables, provide me the result of the Cardano domain belonging:

<br/>
<br/>
```raku
multi sub is-cardano-triplet( $a, $b, $c ) {

    my $left  = .sign * .abs**( 1 / 3 ) given ( $a + $b * $c.sqrt );
    my $right = .sign * .abs**( 1 / 3 ) given ( $a - $b * $c.sqrt );
    return 1 =~= ( $left + $right );
}


multi sub is-cardano-triplet( @triplet ) {
    return is-cardano-triplet( @triplet[ 0 ], @triplet[ 1 ], @triplet[ 2 ] );
}

```
<br/>
<br/>

The second implementation of the `is-cardano-triplet` function is only a placeholder to simplify when I use an array, and this will become clearer later. However, I could have simply used the `|@triplet` flatting operator.

<br/>
The `$left` and `$right` are computed as the parts of the equation. The `.sign` method provides `+1` or `-1` depending on the sign of the result (the part in the `given`) and then I computer the third root of the absolute value, thus providing the resulting signed value. I needed a little help here about the math.
<br/>
However, I was expecting this resulted in an integer result, but instead the result was a `Rat`, thus requiring me to use the new (to me) `=~=` operator, that is *almost equal* operator (see [the documentation](https://docs.raku.org/routine/=~=){:target="_blank}).
<br/>
With all in place, I used a nested loop to generate a triplet, and then I compared every single permutation of the triple to see if the result was good enough:


<br/>
<br/>

``` raku
sub MAIN( Int $limit = 5 ) {
    my @triplets = lazy gather {
        for 1 .. Inf -> $a {
            for 1 ..^ $a -> $b {
                for 1 ..^ $b -> $c {
                    $_.take if is-cardano-triplet( $_ ) for ( $a, $b, $c ).permutations;

                }
            }
        }
    };

    @triplets[ 0 .. $limit ].join( "\n" ).say;
}

```
<br/>
<br/>

I used a `lazy gather` to stop the computation as soon as the end is reached, and in fact I print out only the first `$limit` entries.
This took near 7 seconds on my machine.


<a name="task1pg"></a>
## PWC 148 - Task 1 in PostgreSQL

The implementation, bit to bit, of the Raku solution for this task:

<br/>
<br/>

``` sql
SELECT v
FROM generate_series( 1, 10 ) v
WHERE
v IN ( 2, 4, 6 )

UNION

SELECT v
FROM generate_series( 11, 19 ) v
WHERE v IN ( 12 )

UNION

SELECT v
FROM generate_series( 20, 100 ) v
WHERE
v % 10 IN ( 2, 4, 6 )
AND ( v / 10 )::int  IN ( 3, 4, 5, 6 );

```
<br/>
<br/>

A single query can do the trick.


<a name="task2pg"></a>
## PWC 148 - Task 2 in PostgreSQL

A recursive CTE implementation of the Raku solution for this task, limiting the numbers to `30` to avoid too much joins.
Note that I search for the triplets that provide a sum that is near `1` by a small value.

<br/>
<br/>
```sql
WITH RECURSIVE
triplets AS
(
        SELECT a::numeric, b::numeric, c::numeric
        FROM generate_series( 1, 30 ) a
             , generate_series( 1, 30 ) b
             , generate_series( 1, 30 ) c
        ORDER BY a, b, c
)
, cardano_sum AS
(
        SELECT a, b, c,
               ( a + b * sqrt( c ) )   AS l
               ,( a - b * sqrt( c ) )  AS r
               FROM triplets
)
, cardano AS
(
        SELECT a, b, c, l, r
               , CASE WHEN l < 0 THEN -1 ELSE 1 END * pow( abs( l )::numeric, 1/3::numeric )
               + CASE WHEN r < 0 THEN -1 ELSE 1 END * pow( abs( r )::numeric, 1/3::numeric )
               AS triplet_sum
               FROM cardano_sum
)

SELECT *
FROM cardano
WHERE
abs( 1 - triplet_sum::numeric ) <= 0.0000000001
LIMIT 5
;

```
<br/>
<br/>


It takes 54 seconds to complete, and is a lot more of the Raku implementation. Why? Because the above query performs the computation for every join available.
