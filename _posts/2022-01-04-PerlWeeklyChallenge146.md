---
layout: post
title:  "Perl Weekly Challenge 146: the first challenge of the year!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 146: the first challenge of the year!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 146](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0146/){:target="_blank"}.

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
## PWC 146 - Task 1
The first task was about computing the `1001`-nth prime number, and I decided to implement it by storing the prime numbers into an array, of course lazily, and then pick up the selected value:


<br/>
<br/>
```raku
sub MAIN( Int $which where { $which > 0 } = 1001 ) {
    my @primes = lazy { ( 1 .. Inf ).grep( *.is-prime ); }
    @primes[ $which ].say;
}

```
<br/>
<br/>

Quite simple and staightforward.



<a name="task2"></a>
## PWC 146 - Task 2

The second task was about implementing a search script against a *fraction tree*.
I decided to implement first a `Node` class to represent a single element of the tree:

<br/>
<br/>
```raku
class Node {
    has Rat $.member;
    has Int $.level;
    has Node $.left;
    has Node $.right;
    has Node $.parent is rw;

    submethod BUILD( Rat :$member, Int :$level = 1, Int :$stop-at = 4 ) {
        $!member = $member;
        $!level  = $level;

        if ( $level < $stop-at ) {
            my $sum = $!member.numerator + $!member.denominator;
            $!left  = Node.new: member => $member.numerator / $sum,
                      level => $level + 1,
                      stop-at => $stop-at;
            $!right = Node.new: member => $sum / $member.denominator,
                      level => $level + 1,
                      stop-at => $stop-at;
        }
    }


    method adjust() {
        $!left.parent  = self if $!left;
        $!right.parent = self if $!right;
        $!left.adjust if $!left;
        $!right.adjust if $!right;
    }

    method search-from-here ( Rat $needle ) {
        return self if $!member ~~ $needle;

        if ( $!left ) {
            my $left = $!left.search-from-here( $needle );
            return $left if $left;
        }
        if ( $!right ) {
            my $right = $!right.search-from-here( $needle );
            return $right if $right;
        }
        return Nil;
    }

    method Str(){ $!member.numerator ~ '/' ~ $!member.denominator }
}


```
<br/>
<br/>
When a `Node` is built, it is attached to its children, `$!left` and `$!right`, at least up to a specific deep level.
The `adujust` method links back every children to its parent, because this cannot be done during the construction of the tree since the parent is still not a `Node`.
The `search-from-here` method is the core of the program, and does a deep-first approach in searching for the specified `$needle` across a node, its left children and then right children.
Therefore, the final program becomes:

<br/>
<br/>

``` raku
sub MAIN( Rat $member ) {
    my $level = 1;
    my $root = Node.new: member => 1.Rat;
    $root.adjust;


    my Node $which = $root.search-from-here( $member );
    "Not found $member " and exit if ! $which;
    "Node $member found: { $which.Str } with parent { $which.parent.Str } and grandparent { $which.parent.parent.Str }".say;
}

```
<br/>
<br/>

First the root node is built, and then `adjust` is invoked to fully backlink every node.
Then the `$which` node is the result of searching from the root node into the tree, and the remaining is just printing out the result.

<a name="task1pg"></a>
## PWC 146 - Task 1 in PostgreSQL

The PostgreSQL implementation is made up by two parts:
- a function `f_is_prime` to check if a value is prime;
- a recursive CTE to scan over all primes and select the wanted one.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
f_is_prime( val int )
RETURNS bool
AS $CODE$
DECLARE
        i int;
BEGIN
     IF val <= 0 THEN
        RAISE EXCEPTION 'Cannot use a number less than 1!';
     END IF;

     FOR i IN 2 .. ( val - 1 ) LOOP
         IF val % i = 0 THEN
            RETURN false;
         END IF;
     END LOOP;

     RETURN true;
END
$CODE$
LANGUAGE plpgsql;


WITH primes AS (
     SELECT n as needle, row_number() OVER( PARTITION BY f_is_prime( n ) ) as idx
     FROM generate_series( 1, 10000 ) n
     WHERE f_is_prime( n )
     ORDER BY n
)
SELECT *
FROM primes
WHERE idx = 1001;


```
<br/>
<br/>

The recusrive CTE is the interesting part. The `primes` materializes all primes numbers less than `10000`. I use a window function `row_number` to count the position of each prime number.
Then I select exactly one row, that is the one with the count equal to `1001`.


<a name="task2pg"></a>
## PWC 146 - Task 2 in PostgreSQL

The second task has been a little more complicated to implement in PostgreSQL. I decided to got with the following approach:
- create a table that represent the `Node` structure;
- populate the table by means of a function that adds another level to the tree;
- create a function to search for within the tree, and that returns three descriptive tuples.

<br/>
Let's start with the table structure:

<br/>
<br/>
```sql
DROP TABLE IF EXISTS fraction_tree;
CREATE TABLE fraction_tree (
       pk int GENERATED ALWAYS AS IDENTITY
       , numerator int default 1
       , denominator int default 1
       , child_of int
       , level int default 1
       , PRIMARY KEY( pk )
       , FOREIGN KEY (child_of) REFERENCES fraction_tree( pk )
);


TRUNCATE TABLE fraction_tree;
ALTER TABLE fraction_tree ALTER COLUMN pk RESTART;


INSERT INTO fraction_tree( numerator, denominator )
VALUES( 1, 1 );

```
<br/>
<br/>

The `fraction_tree` table is always "resetted" to the only one value of the root.
Then it comes the function to add a single level to the tree:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_add_one_level_fraction_tree()
RETURNS INT
AS $CODE$
DECLARE
        current_left   fraction_tree%rowtype;
        current_right  fraction_tree%rowtype;
        previous_tuple fraction_tree%rowtype;
        nodes_added    int := 0;
BEGIN

        FOR previous_tuple IN SELECT * FROM fraction_tree
                                     WHERE level = ( SELECT max( level ) FROM fraction_tree )
                                     LOOP


                current_left.numerator   := previous_tuple.numerator;
                current_left.denominator := ( previous_tuple.numerator + previous_tuple.denominator );
                current_left.child_of    := previous_tuple.pk;
                current_left.level       := previous_tuple.level + 1;
                current_left.pk          := nextval( 'fraction_tree_pk_seq' );

                current_right.numerator   := ( previous_tuple.numerator + previous_tuple.denominator );
                current_right.denominator := previous_tuple.denominator;
                current_right.child_of    := previous_tuple.pk;
                current_right.level       := previous_tuple.level + 1;
                current_right.pk          := nextval( 'fraction_tree_pk_seq' );

                INSERT INTO fraction_tree
                OVERRIDING SYSTEM VALUE
                SELECT current_left.*;

                INSERT INTO fraction_tree
                OVERRIDING SYSTEM VALUE
                SELECT current_right.*;

                nodes_added := nodes_added + 2;

       END LOOP;

       RETURN nodes_added;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Every node has its level, so I first select the nodes on the highest level (the leaves), and then iterate on each of them assigning their values to `previous_tuple`. For every node found, I do generate its left and right node and insert them into the table.

But doing this population manually is boring, so I placed also a wrapper function that populates the table with the specified depth:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_populate_fraction_tree( levels int default 4 )
RETURNS int
AS $CODE$
DECLARE
        i           int := 0;
        nodes_added int := 0;
BEGIN
     FOR i IN 1 .. levels LOOP
         nodes_added := nodes_added + f_add_one_level_fraction_tree();
     END LOOP;

     RETURN nodes_added;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


Last comes the function to find out a node, its parent and its grandparent. I decided to output a kind of text description of every node, but it is possible to change the result set to whatever information you need.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_search_for_fraction_tree( numer int, denomin int )
RETURNS TABLE ( description text, fraction text )
AS $CODE$
DECLARE
        current_tuple fraction_tree%rowtype;
BEGIN
        SELECT 'child', numerator || '/' || denominator
        INTO description, fraction
        FROM fraction_tree
        WHERE numerator   = numer
        AND   denominator = denomin;

        IF FOUND THEN
           RETURN NEXT;


           SELECT 'parent', numerator || '/' || denominator
           INTO description, fraction
           FROM fraction_tree
           WHERE pk = ( SELECT child_of
                        FROM fraction_tree
                        WHERE numerator   = numer
                        AND   denominator = denomin );

           RETURN NEXT;



           SELECT 'grandparent', numerator || '/' || denominator
           INTO description, fraction
           FROM fraction_tree
           WHERE pk = ( SELECT child_of
                        FROM fraction_tree
                        WHERE pk = ( SELECT child_of
                                     FROM fraction_tree
                                     WHERE numerator   = numer
                                     AND   denominator = denomin ) );

          RETURN NEXT;

        END IF;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The idea is simple: I do search for the current node, and if I found it, I do search for its parent and grandparent. I could have done it with a recursive CTE or some other way, but this seems a quite simple approach.
