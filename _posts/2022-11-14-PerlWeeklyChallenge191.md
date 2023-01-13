---
layout: post
title:  "Perl Weekly Challenge 191: permutations!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 191: permutations!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 191](https://perlweeklychallenge.org/blog/perl-weekly-challenge-191/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)

<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)


Last, the solutions in PostgreSQL PL/PgSQL:

<br/>
- [Task 1 in PostgreSQL Pl/PgSQL](#task1plpgsql)
- [Task 2 in PostgreSQL Pl/PgSQL](#task2plpgsql)



<a name="task1"></a>
## PWC 191 - Task 1 - Raku Implementation

The first task was about finding out if, in a list of integers, all the items are less than half the size of the greatest one.

<br/>
<br/>
```raku
sub MAIN( *@l where { @l.grep( * ~~ Int ).elems == @l.elems } ) {
    my $max = @l.max;
    my @ll = @l.grep: { $_ == $max ||  $_ * 2 <= $max };
    '1'.say and exit if @ll.elems == @l.elems;
    '-1'.say;
}

```
<br/>
<br/>

The idea is quite simple: given the input list `@l` I search for the `$max`, and then do a `grep` on the list itself. The `grep` counts all the elements where the value is exactly the `$max` found or the element multiplied by `2` is less the `$max`.
If the final list `@ll` has the same number of elements as the starting one, than the result is ok, otherwise it is not.


<a name="task2"></a>
## PWC 191 - Task 2 - Raku Implementation

Given a number `n` see how many *cute lists* can be produced. A cute list is one where either:
- the element at position `i` is evenly divisble by `i`
- the index `i` is evenly divisible by element at index `i`.

<br/>
<br/>
```raku
sub MAIN( Int $n where { 0 < $n <= 15 } ) {

    my $cute-counter = 0;
    for ( 1 .. $n ).List.permutations -> $current-list {
	my $is-cute = True;
	for 0 ..^ $current-list.elems -> $i {
	    $is-cute = False and last if $current-list[ $i ] !%% ( $i + 1 );
	}

	$cute-counter++ if $is-cute;

	$is-cute = True;
	for 0 ..^ $current-list.elems -> $i {
	    $is-cute = False and last if ( $i + 1 ) !%% $current-list[ $i ]  ;
	}

	$cute-counter++ if $is-cute;

    }
    $cute-counter.say;
}

```
<br/>
<br/>

I do permute on all possible lists, and for every `$current-list` at every step, I see if one or the other condition is satisfied, stopping the search at the very first element that does not satisfy the condition.


<a name="task1plperl"></a>
## PWC 191 - Task 1 - PL/Perl Implementation

This is roughly the same implementation as Raku:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc191.task1_plperl( int[] )
RETURNS int
AS $CODE$
   my ($l) = @_;

   my $max = 0;

   # compute the max element
   for ( $l->@* ) {
       $max = $_ if ( $max < $_ );
   }

   # iterate on all elements and see
   # if one of the is doubly greater than the max
   for ( $l->@* ) {
       next if $_ == $max;
       return -1 if $_ * 2 > $max;
   }

   return 1;
$CODE$
LANGUAGE plperl;


```
<br/>
<br/>

This time I didn't use a `grep`, rather a common loop to check every element. Why? No special reason, it simply came out of my mind!



<a name="task2plperl"></a>
## PWC 191 - Task 2 - PL/Perl Implementation

Here I decided to use `List::Permute::Limit` in order to get all the possible permutations of the input list.
The rest of the code is pretty much the same as in the Raku implementation.


<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION
pwc191.task2_plperl( int )
RETURNS int
AS $CODE$
   use List::Permute::Limit qw(permute_iter permute);

   my ($n) = @_;
   my $cute_counter = 0;
   my @l = ( 1 .. $n );

   my @permutations = permute( items => [ @l ], nitems => $n );
   for my $current_list ( @permutations ) {

       my $is_cute = 1;
       for my $i ( 0 .. $current_list->@* ) {

       	   if ( $current_list->[ $i ] % ( $i + 1 ) != 0 ) {
	      $is_cute = 0;
	      last;
	   }
       }

       $cute_counter++ if ( $is_cute );

       $is_cute = 1;
       for my $i ( 0 .. $current_list->@* ) {

       	   if ( ( $i + 1 ) % $current_list->[ $i ]  != 0 ) {
	      $is_cute = 0;
	      last;
	   }
       }

       $cute_counter++ if ( $is_cute );
   }

   return $cute_counter;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>




<a name="task1plpgsql"></a>
## PWC 191 - Task 1 - PL/PgSQL Implementation

The task can be solved with a couple of queries:

<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION
pwc191.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
	current_max int;
	wrong int := 0;
BEGIN
	-- compute the max
	SELECT max( v )
	INTO   current_max
	FROM unnest( l ) v;

	SELECT count(*)
	INTO   wrong
	FROM unnest( l ) v
	WHERE ( v * 2 ) > current_max
	AND   v <> current_max;

	IF wrong > 0 THEN
	   RETURN -1;
        ELSE
          RETURN 1;
        END IF;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The first `unnest` query converts the input array into a table, so that I can use the `max` operator to compute the value. The second `unnest` query extracts all the values that, once doubled, are greater than the current max and counts them. Therefore, if the counting of `wrong` tuples is greater than zero, the array is not good.


<a name="task2plpgsql"></a>
## PWC 191 - Task 2 - PL/PgSQL Implementation

This has been a lot more difficult than all the other implementations, since permuting a list in PostgreSQL is not as simple as it may sound.

<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION
pwc191.task2_plpgsql( n int )
RETURNS int
AS $CODE$
DECLARE
	cute_counter int := 0;
	i int;
	src int[];
	permutation int[];
	is_cute bool;
BEGIN

	FOR i IN 1 .. n LOOP
	    src = src || i;
	END LOOP;




	FOR permutation IN
	with recursive
	data as (select src as arr),
        keys as (select generate_subscripts(d.arr, 1) as rn from data d),
        cte  as (
           select d.arr initial_arr, array[d.arr[k.rn]] new_arr, array[k.rn] used_rn
           from data d
           cross join keys k
           union all
           select initial_arr, c.new_arr || c.initial_arr[k.rn], used_rn || k.rn
           from cte c
           inner join keys k on not (k.rn = any(c.used_rn))
    )
    select new_arr
    from cte
    WHERE array_length( new_arr, 1 ) = n
    LOOP


	 is_cute := 1;
	 FOR i IN 1 .. array_length( permutation, 1 ) LOOP
	 	 IF permutation[i] % i <> 0  THEN
		    is_cute = false;
		    EXIT;
		 END IF;
	 END LOOP;

	 IF is_cute THEN
	    cute_counter := cute_counter + 1;
	 END IF;

	 is_cute := 1;
	 FOR i IN 1 .. array_length( permutation, 1 ) LOOP
	 	 IF i % permutation[i] <> 0  THEN
		    is_cute = false;
		    EXIT;
		 END IF;
	 END LOOP;
	 IF is_cute THEN
		 cute_counter := cute_counter + 1;
	 END IF;

    END LOOP;


RETURN cute_counter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

First of all, I compute the starting array `src` by creating the sequence of numbers.
Then I do a `FOR` loop with a huge recursive CTE that computes all possible permutations of the array, taking into account only the arrays with the size equal to `n`.
For every current permutation, named `permutation`, I iterate on every element and see if any of the two required conditions are met.
The `cute_counter` is incremented accordingly depending if the conditions are met or not.
