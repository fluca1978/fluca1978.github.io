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
In the following, the assigned tasks for [Challenge 154](https://perlweeklychallenge.org/blog/perl-weekly-challenge-154/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for other PWC in the past, I've done also a couple of possible implementations in PostgreSQL:
<br/>
- [Task 1 in PostgreSQL](#task1pg)
- [Task 2 in PostgreSQL](#task2pg)

<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)

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
The task required to get the first ten unique and prime Padovan numbers, that means I don't know how many Padovan numbers I have to compute, and this sounds good for lazyness:

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
        while ( @unique-padovan-numbers.grep( $current ) || ! $current.is-prime  ) {
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

As you can see, the function is really simple. It is simple also the function to check if a number is prime:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc154.is_prime( n int )
RETURNS bool
AS $CODE$
DECLARE
    i int;
BEGIN
    FOR i IN 2 .. n - 1 LOOP
        IF n % i = 0 THEN
           RETURN false;
        END IF;
    END LOOP;

    RETURN true;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Then I used the functions in a CTE, not recursive, with a limit of `50` numbers to be generated. That's because I already know that it is a safe horizon to get the expected results.

<br/>
<br/>

``` sql
WITH
padovan AS (
   SELECT n, pwc154.padovan( n ) AS p
   FROM generate_series( 0, 50 ) n
)
, padovan_prime
AS (
  SELECT p, pwc154.is_prime( p ) AS prime
  FROM padovan p
)
SELECT distinct( p )
FROM padovan_prime
WHERE prime
ORDER BY 1
LIMIT 10
;
```
<br/>
<br/>

I join the function with `generate_series` to get `51` values, and then I simply do a `SELECT distinct` to get the unique values. I limit the result to `10`, as asked by the task and order ascending.


<a name="task1plperl"></a>
## PWC 154 - Task 1 in PostgreSQL Pl/Perl

The idea is to mix and match some SQL stuff with some Perl stuff. I create a table that contains all the excluded permutations, and that is the easy part. Than I declare a Perl function to build up all the permutations of the given string. To achieve this I used `List::Permutor`, that has to be installed as a module so that PostgreSQL can find it.
Last, a function `find_missing_permutations` builds up an SQL query to mimic the SQL implementation, so to exclude from all the permutations the content of the table.

<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc154;

CREATE TABLE IF NOT EXISTS
pwc154.permutations
(
  pk int generated always as identity
  , perm text not null
  , primary key( pk )
);

TRUNCATE pwc154.permutations;

INSERT INTO pwc154.permutations( perm )
VALUES
( 'PELR' ),
( 'PREL' ),
( 'PERL' ),
( 'PRLE' ),
( 'PLER' ),
( 'PLRE' ),
( 'EPRL' ),
( 'EPLR' ),
( 'ERPL' ),
( 'ERLP' ),
( 'ELPR' ),
( 'ELRP' ),
( 'RPEL' ),
( 'RPLE' ),
( 'REPL' ),
( 'RELP' ),
( 'RLPE' ),
( 'RLEP' ),
( 'LPER' ),
( 'LPRE' ),
( 'LEPR' ),
( 'LRPE' ),
( 'LREP' )
;


CREATE OR REPLACE FUNCTION pwc154.all_permutations( text )
RETURNS SETOF text
AS $CODE$
 use List::Permutor;
 my @letters = split( //, $_[ 0 ] );
 my $engine  = List::Permutor->new( @letters );
 while ( my @permutation = $engine->next() ) {
    my $current = join( '', @permutation );
    return_next( join( '', @permutation ) );
 }

 return undef;
$CODE$
LANGUAGE plperlu;


CREATE OR REPLACE FUNCTION pwc154.find_missing_permutations( text )
RETURNS SETOF text
AS $CODE$
  elog( INFO, "SELECT perm FROM pwc154.all_permutations( $_[ 0 ] ) WHERE perm NOT IN ( SELECT perm FROM pwc154.permutations )" );
  my $result_set = spi_exec_query( "SELECT perm FROM pwc154.all_permutations( " . quote_literal( $_[ 0 ] ) . " ) t(perm) WHERE perm NOT IN ( SELECT perm FROM pwc154.permutations )" );
  for my $i ( 0 .. $result_set->{ processed } ) {
      return_next( $result_set->{ rows }[ $i ]->{ perm } );
  }
  return undef;
$CODE$
language plperl;

```
<br/>
<br/>

Note that in order to use an external module, the `all_permutations` function must be run as `plperlu`, that means *untrusted* language within PostgreSQL.



<a name="task2plperl"></a>
## PWC 154 - Task 2 in PostgreSQL Pl/Perl

The second task is based on a pile of Perl functions: one computes a single Padovan number using recursion via SQL; the seconda builds all the Padovan numbers up to a given limit. Last an SQL query extracts the first `10` unique numbers.


<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc154;


CREATE OR REPLACE FUNCTION
pwc154.padovan_plperl(  int )
RETURNS int
AS $CODE$
 return 1 if $_[0] <= 2;
 my ( $a, $b ) = ( $_[ 0 ] - 3, $_[ 0 ] - 2 );
 my $rs = spi_exec_query( "SELECT pwc154.padovan_plperl( $a ) + pwc154.padovan_plperl( $b ) AS p" );
 return $rs->{ rows }[ 0 ]->{ p };

$CODE$
LANGUAGE plperl;

CREATE OR REPLACE FUNCTION
pwc154.plperl_is_prime( int )
RETURNS bool
AS $CODE$
   for my $i ( 2 .. ( $_[0] - 1 ) ) {
      return 0 if $_[0] % $i == 0;
   }

  return 1;
$CODE$
LANGUAGE plperl;


CREATE OR REPLACE FUNCTION
pwc154.padovans_up_to( int )
RETURNS SETOF int
AS $CODE$
 for my $i ( 0 .. $_[ 0 ] ) {
     my $rs = spi_exec_query( "SELECT pwc154.padovan_plperl( $i ) AS p" );
     return_next(  $rs->{ rows }[ 0 ]->{ p } );
 }

 return undef;
$CODE$
LANGUAGE plperl;


SELECT distinct( p.p )
FROM pwc154.padovans_up_to( 50 ) p
WHERE pwc154.plperl_is_prime( p.p ) = true
ORDER BY 1
LIMIT 10;

```
