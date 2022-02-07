---
layout: post
title:  "Perl Weekly Challenge 151: trees and robberies"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 151: trees and robberies

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 151](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0151/){:target="_blank"}.

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
## PWC 151 - Task 1

I hate trees!
<br/>
The task provided a stringified version of a tree, then it was asked to find the shortest path to a leaf. The problematic part was to build the tree starting from the string.
<br/>
I created a simple `Node` class:


<br/>
<br/>
```raku
class Node {
    has Int $.value;
    has Node $.left is rw;
    has Node $.right is rw;
    has Node $.parent is rw;
}

 ```
<br/>
<br/>

Then, the string must be translated into a workable tree, that I placed into a `@tree` aray where each tree level is an array of nodes:

<br/>
<br/>

``` raku
sub MAIN( Str $nodes, Str $delim = '|' ) {
    my @nodes-as-text = $nodes.split( $delim );
    my @tree;


    for @nodes-as-text -> $current-nodes {
        my @current-level;
        my $index = -1;
        my @values = $current-nodes.words;
        @values.push: '*' if @values.elems !%% 2;
        for @values -> $left, $right {
            say "$left, $right";
            $index++;
            my Node $left-node  = Node.new: value => $left.Int  if $left !~~ '*';
            my Node $right-node = Node.new: value => $right.Int if $right !~~ '*';

            @current-level.push: $left-node  if $left-node;
            @current-level.push: $right-node  if $right-node;

            next if ! @tree;

            @tree[ * - 1 ][ $index ].left  = $left-node  if $left-node;
            $left-node.parent = @tree[ * - 1 ][ $index ] if $left-node;

            @tree[ * - 1 ][ $index ].right = $right-node if $right-node;
            $right-node.parent = @tree[ * - 1 ][ $index ] if $right-node;


        }

        @tree.push: @current-level;
    }


    my @paths;
    for @tree.reverse -> @leaves {
        for @leaves -> $current-leaf is rw {
            next if $current-leaf.left || $current-leaf.right;
            my $path = 0;
            while ( $current-leaf ) {
                $path++;
                $current-leaf = $current-leaf.parent;
            }
            @paths.push: $path;
        }
    }

    @paths.min.say;
}

```
<br/>
<br/>


Once the `@tree` array is built, I start from the `@leaves`, the last level of the array, and start to count going upper.
Every path count is placed into `@paths`, so that I can quickly compute the `min`.



<a name="task2"></a>
## PWC 151 - Task 2

Robbering houses in a row skipping an house after another:

<br/>
<br/>
```raku
sub MAIN( Str $houses, Bool $verbose = True ) {
    my %rob;
    my @values = $houses.words;
    my $current-index = 0;



    # bootstrap

    %rob{ $current-index } = @values[ $current-index ];




    while ( $current-index < ( @values.elems - 2 ) ) {
        my %current;
        %current = key => $_,
                   value => @values[ $_ ]
                         if ! %current || @values[ $_ ] > %current<value>
                         for ( $current-index + 2 ) ..^ @values.elems;

        %rob{ %current<key> } = %current<value>;
        $current-index = %current<key>;
    }

    # print the result
    %rob.values.sum.say;

    # print the status
    if $verbose {
        "House { .key } = { .value } ".say for %rob;
    }
}

```
<br/>
<br/>

I use a `%rob` hash with the key as the index of the houses, and the value the robbery gain.
Then I use a `Pair` to store the max value available from remaining houses, skipping the next to the current one.
At the end, I print the `sum` of the robbery gain.


<a name="task1pg"></a>
## PWC 151 - Task 1 in PostgreSQL

Implementing the first task in PostgreSQL was a little long. First of all, I needed a table to represent the `Node` of the tree, where each row is a node of the tree. Then there is the need for a *parsing* of the stringified version of the tree. Therefore, alongside the tables, I created a couple of functions:
- `f_flat_tree` accepts the string and provides a table like node structures with values and tree level;
- `f_generate_tree` is a `PROCEDURE` and takes the output of the previous function and populates the `node` table accordingly.

<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc151;

CREATE TABLE IF NOT EXISTS pwc151.node(
       pk int generated always as identity
       , val int
       , parent_pk int
       , left_pk int
       , right_pk int
       , level int
       , PRIMARY KEY( pk )
       , FOREIGN KEY( parent_pk ) REFERENCES pwc151.node( pk )
       , FOREIGN KEY( left_pk ) REFERENCES pwc151.node( pk )
       , FOREIGN KEY( right_pk ) REFERENCES pwc151.node( pk )
);

TRUNCATE pwc151.houses CASCADE;

/**
 * Produces a flat tree like:

testdb=> SELECT * FROM  pwc151.f_flat_tree( '1 | 2 3 | 4 5 ');
DEBUG:  Inspecting [1]
DEBUG:  Inspecting [|]
DEBUG:  Changing level!
DEBUG:  Inspecting [2]
DEBUG:  Inspecting [3]
DEBUG:  Inspecting [|]
DEBUG:  Changing level!
DEBUG:  Inspecting [4]
DEBUG:  Inspecting [5]
val | lev
-----+-----
1 |   0
2 |   1
3 |   1
4 |   2
5 |   2
(5 rows)
*/
CREATE OR REPLACE FUNCTION
pwc151.f_flat_tree( tree text, delim char default '|' )
RETURNS TABLE( val int, lev int )
AS $CODE$
DECLARE
        needle text;
BEGIN
        lev := 0;
        FOR needle IN SELECT v FROM regexp_split_to_table( tree, delim ) v LOOP
            CONTINUE WHEN needle ~ '^\s+$';
            RAISE DEBUG 'Inspecting [%]', needle;

            IF needle = delim THEN
               RAISE DEBUG 'Changing level!';
               lev := lev + 1;
               CONTINUE;
           END IF;

           IF needle = '*' THEN
              val := NULL;
           ELSE
              val := needle::int;
           END IF;

           RETURN NEXT;
        END LOOP;
    RETURN;
END
$CODE$
LANGUAGE plpgsql;








CREATE OR REPLACE PROCEDURE
pwc151.f_generate_tree( tree text, delim text default '|' )
AS $CODE$
DECLARE
        current_tuple pwc151.node%rowtype;
        parent_tuple  pwc151.node%rowtype;
        lev int := 1;
        val int := 0;
        max_lev int;
        counter int := 0;
        last_pk int := -1;
BEGIN
        TRUNCATE pwc151.node CASCADE;

        INSERT INTO pwc151.node( val, level )
        SELECT * FROM pwc151.f_flat_tree( tree, delim );

        COMMIT;

        SELECT max( level )
        INTO max_lev
        FROM pwc151.node;

        RAISE DEBUG 'Max level found %', lev;

        WHILE lev <= max_lev LOOP
              RAISE DEBUG '========= Level % ==============', lev;


           FOR parent_tuple IN SELECT * FROM pwc151.node WHERE level = lev - 1 ORDER BY pk LOOP
                  counter := 0;
              FOR current_tuple IN SELECT * FROM pwc151.node
                                   WHERE level = lev
                                   AND parent_pk IS NULL
                                   AND pk >= last_pk
                                   ORDER BY pk LOOP
                  last_pk := current_tuple.pk;
                  EXIT WHEN counter >= 2;
                  RAISE DEBUG 'Step %: Adjusting node [%] [%]', counter, current_tuple.val, current_tuple.pk;
                  CONTINUE WHEN lev = 0; -- root node


                      RAISE DEBUG 'Parent node [%] -> [%] = [%]', parent_tuple.val, ( counter % 2 = 0 ), current_tuple.val;
                      counter := counter + 1;

                      CONTINUE WHEN current_tuple.val IS NULL;

                      IF counter % 2 <> 0 THEN
                         RAISE DEBUG 'Left node [%] child of [%]', current_tuple.val, parent_tuple.val;
                         UPDATE pwc151.node
                         SET    left_pk = current_tuple.pk
                         WHERE  pk = parent_tuple.pk;
                      ELSE
                         RAISE DEBUG 'Right node [%] child of [%]', current_tuple.val, parent_tuple.val;
                        UPDATE pwc151.node
                        SET    right_pk = current_tuple.pk
                        WHERE  pk = parent_tuple.pk;
                      END IF;

                      UPDATE pwc151.node
                      SET    parent_pk = parent_tuple.pk
                      WHERE  pk = current_tuple.pk;

                      COMMIT;


                  END LOOP;
             END LOOP;

             lev := lev + 1;
        END LOOP;
END
$CODE$
LANGUAGE plpgsql;




WITH RECURSIVE min_depth AS
(
        -- root
        SELECT pk, val, level, 'root' as description, 1 as depth
        FROM pwc151.node
        WHERE level = 0
        AND parent_pk IS NULL

        UNION

        SELECT n.pk, n.val, n.level, description || '->' || n.val, p.depth + 1
        FROM  min_depth p, pwc151.node n
        WHERE n.level = p.level + 1
        AND   n.parent_pk = p.pk
)
SELECT description, depth
FROM min_depth
WHERE depth = ( SELECT min( depth ) FROM min_depth WHERE level > 0 );

```
<br/>
<br/>

Last, a *recursive CTE* does the trick of finding out all the shortest paths and their length.


<a name="task2pg"></a>
## PWC 151 - Task 2 in PostgreSQL

The second task is pretty much a re-implementation of the Raku solution with a table, named `houses` that stores the values.

<br/>
<br/>
```sql
CREATE SCHEMA IF NOT EXISTS pwc151;

CREATE TABLE IF NOT EXISTS pwc151.houses(
       pk int generated always as identity
       , index int
       , value int
       , PRIMARY KEY( pk )
);


TRUNCATE pwc151.houses;
INSERT INTO pwc151.houses( index, value )
VALUES ( 0, 4), ( 1, 2 ), (2, 3), (4, 6), (5,5),(6,3);


CREATE OR REPLACE FUNCTION
pwc151.f_sum()
RETURNS int
AS $CODE$
DECLARE
        current_row pwc151.houses%rowtype;
        s int := 0;
        max_index int;
BEGIN
        -- bootstrap
        SELECT *
        INTO current_row
        FROM pwc151.houses
        ORDER BY index
        LIMIT 1;

        s := s + current_row.value;


        SELECT max( index )
        INTO max_index
        FROM pwc151.houses;

        WHILE current_row.index < max_index LOOP
              SELECT *
              INTO current_row
              FROM pwc151.houses
              WHERE index > current_row.index + 1
              ORDER BY value DESC
              LIMIT 1;

              s := s + current_row.value;

        END LOOP;

        RETURN s;
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>

The function `f_sum` iterates on every row found with the index within the right boundaries and computes the max value using the trick of ordering by `value` and limiting the result set to the first row, so to get the max value without the need for a subquery.
