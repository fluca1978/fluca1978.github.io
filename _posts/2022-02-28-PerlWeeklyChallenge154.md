---
layout: post
title:  "Perl Weekly Challenge 154: lazyness and recursion"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 154: lazyness and recursion

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 154](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0154/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for other PWC in the past, I've done also a couple of possible implementations in PostgreSQL:
<br/>
- [Task 1 in PostgreSQL](#task1pg)
- [Task 2 in PostgreSQL](#task2pg)

<a name="task1"></a>
## PWC 154 - Task 1

A one liner! Well, not really a one liner because of the whistles and bells to check for the arguments, but the implementation is really a single line. The task was asking to search for any missing permutation of the letters of an input words hat was not already contained in a list of permutations provided as input.
<br/>



<br/>
<br/>
```raku
sub MAIN( Str $needle = 'PERL',
              @input =  qw/ PELR PREL PERL PRLE PLER PLRE EPRL EPLR ERPL
                         ERLP ELPR ELRP RPEL RPLE REPL RELP RLPE RLEP
                         LPER LPRE LEPR LRPE LREP /  ) {

    $needle.comb.permutations.map( { $_.join unless @input.grep( $_.join ) } ).join( "\n" ).say;
}
 ```
<br/>
<br/>

The idea is really simple:
- I do `comb` the input word so to get an array of letters;
- then I compute the `permutations`;
- then `map` the array of permutations extracting only those words (`$_.join`) that are not contained `grep` in the `@input` permutations array;
- then `join` and `say` the result.


<a name="task2"></a>
## PWC 154 - Task 2

*Padovan numbers*, something I was not aware of. Essentially, a Padovan number `P(n)` is made by the sum of `P(n-3) + P(n-2)` with the bootstrap terms `P(0) = P(1) = P(2) =1`.
The task required to get the first ten unique Padovan numbers, that means I don't know how many Padovan numbers I have to compute, and this sounds good for lazyness:

<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit > 0 } = 10 ) {

    my @padovan-numbers = lazy gather {
        for 0 .. Inf {
            # initial values
            take 1 if $_ == any( 0, 1, 2 );
            take @padovan-numbers[ $_ - 3 .. $_ - 2 ].sum if $_ > 2;
        }
    };

    my $current-index = 0;
    my @unique-padovan-numbers;
    while ( @unique-padovan-numbers.elems < $limit ) {
        my $current = @padovan-numbers[ $current-index ];
        while ( @unique-padovan-numbers.grep( $current ) ) {
            $current = @padovan-numbers[ ++$current-index ];
        }

        @unique-padovan-numbers.push: $current;
    }

    @unique-padovan-numbers[ 0 .. $limit ].join( ', ' ).say;
}

```
<br/>
<br/>

The `@padovan-numbers` array will be lazyly initialized with the values depending on the iteration we are on.
Then, I do `push` every Padovan number that is not already contained into `@unique--padovan-numbers` and go looping until I reach the required size of the array, that means the required number of unique numbers.
Last, I print the result.


<a name="task1pg"></a>
## PWC 154 - Task 1 in PostgreSQL

Well, recursive CTEs to the rescue! It is possible to use a recursive CTE to get all the permutations of the given word, and then perform a *simple* `SELECT` to find out the disjoint sets:


<br/>
<br/>

``` sql
WITH RECURSIVE
letters( l ) AS (
        SELECT *
        FROM regexp_split_to_table( 'PERL', '' )
)
, permutations AS
(
        SELECT l, l AS perm, 1 AS level
        FROM letters

        UNION ALL

        SELECT l.l, p.perm || l.l AS perm, level + 1
        FROM letters l, permutations p
        WHERE level <= 100
        AND position( l.l IN p.perm ) = 0
)

SELECT perm
FROM permutations
WHERE length( perm ) = 4
AND perm NOT IN (
'PELR',
'PREL',
'PERL',
'PRLE',
'PLER',
'PLRE',
'EPRL',
'EPLR',
'ERPL',
'ERLP',
'ELPR',
'ELRP',
'RPEL',
'RPLE',
'REPL',
'RELP',
'RLPE',
'RLEP',
'LPER',
'LPRE',
'LEPR',
'LRPE',
'LREP'
)
;

```
<br/>
<br/>

The `letters` part of the query simply provides one row per letter, so that I can then join all the letters in the recursive part named `permutations`. Please note that the CTE is going to provide an increasing in size list of permutations, that means couple of letters, three letters, four and so on depending on the number of rows in `letters`.
That's why, in the outer query, I do filter only on permutations that have a `length` of `4`, as the original string. And ask for all the strings that have no match with the given list.


<a name="task2pg"></a>
## PWC 154 - Task 2 in PostgreSQL

This time I decided to go for recursion: I created a function to provide a given Padovan number.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc154.padovan( i int )
RETURNS int
AS $CODE$
BEGIN
        IF i <= 2 THEN
           RETURN 1;
        END IF;

        RETURN pwc154.padovan( i - 3 ) + pwc154.padovan( i - 2 );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

As you can see, the function is really simple.
<br/>
Then I used the function in a CTE, not recursive, with a limit of `20` numbers to be generated. That's because I already know that it is a safe horizon to get the expected results.

<br/>
<br/>

``` sql
WITH
padovan_20 AS (
   SELECT n, pwc154.padovan( n ) AS p
   FROM generate_series( 0, 20 ) n
)
SELECT distinct( p.p )
FROM padovan_20 p
ORDER BY 1
LIMIT 10;

```
<br/>
<br/>

I join the function with `generate_series` to get `21` values, and then I simply do a `SELECT distinct` to get the unique values. I limit the result to `10`, as asked by the task and order ascending.
