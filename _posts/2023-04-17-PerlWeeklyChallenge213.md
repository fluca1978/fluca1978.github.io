---
layout: post
title:  "Perl Weekly Challenge 213: from here to there!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 213: from here to there!

This post presents my solutions to the [Perl Weekly Challenge 213](https://perlweeklychallenge.org/blog/perl-weekly-challenge-213/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 213 - Task 1 - Raku](#task1)
- [PWC 213 - Task 2 - Raku](#task2)
- [PWC 213 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 213 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 213 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 213 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 213 - Task 1 - Raku Implementation

The first task was really easy: sort a given input list of numbers keeping evens first.

<br/>
<br/>
```raku
sub MAIN( *@n where { @n.elems == @n.grep( { $_ ~~ Int && $_ > 0 } ).elems } ) {
    ( @n.grep( * %% 2 ).sort.join( ',' ) ~ ',' ~ @n.grep( * !%% 2 ).sort.join( ',' ) ).say;
}
```
<br/>
<br/>

It's a single line of code: `grep` the evens, `sort` and concatenate with the same logic for odds.


<a name="task2"></a>
## PWC 213 - Task 2 - Raku Implementation

This is a much more complex task, and I'm not sure my solution is totally correct since I don't know the logic inside a parition.
The task gave you a list of partitions made by nodes, each one identified by a number, and a starting and ending value. You need to find the path from the begin to the end point jumping thru partitions.

<br/>
<br/>
```raku
sub MAIN( Int $source, Int $destination, $r ) {
    my @routes;
    my @current;
    for $r.comb( :skip-empty ) {
	next if ! $_;

	if $_ ~~ "|" {
	    @routes.push: [@current];
	    @current = ();
	    next;
	}

	@current.push: $_.Int if ( $_.Int > 0 );

    }

    @routes.push: [@current] if ( @current );

    my ( $source-index, $destination-index );
    for 0 ..^ @routes.elems -> $index {
		$source-index = $index and next if ( @routes[ $index ].grep( $source ) );
		$destination-index = $index and next if ( @routes[ $index ].grep( $destination ) );
    }


    my @current-path;
    my $next-route = $source-index;
    my $loop = True;
    while ( $loop ) {
	for @routes[ $next-route ].Array -> $node {
	    @current-path.push: $node if ( ! @current-path.grep( * ~~ $node ) );

	    $loop = False;
	    for $next-route ^..^ @routes.elems -> $jump-to {

				if ( @routes[ $jump-to ].grep( { $_ ~~ $node } ) ) {
				    $next-route = $jump-to;
				    $loop = True;
				    last;
				}
	    }

	    last if $loop;
	}
    }

    @current-path.join( ' -> ' ).say;
}

```
<br/>
<br/>

I assume the list of partitions is passed as a string, where each partition is separated by `|`, so the first looping is to get the `@routes` available. Then I search for the `$source-index` that identifies the partition that contains the source point.
From there, I loop over the next partitions, adding every node to `@current-path`. For every node, I scan again the `@routes` to see if there's another partition that contains the current `$node`, and that means is the next partition to jump to.
If I find another partition to jump, I loop again and switch to such partition, until there's no more partitions to visit. I then print the ending result.
<br/>
This is not totally correct, since edge cases, like two partitions having the same next node can screw up the whole thing, as well as a single partition containing both the source and destination nodes...

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 213 - Task 1 - PL/Perl Implementation

Same approach as the Raku solution. Could have been a single line, but for sake of readability I made it count for three:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc213.task1_plperl( int[] )
RETURNS int[]
AS $CODE$
   my ( $array ) = @_;
   my @sorted;
   @sorted = ( sort( grep( { $_ % 2 == 0 } $array->@* ) ),
               sort(  grep( { $_ % 2 != 0 } $array->@* ) ) );
   return [ @sorted ];
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 213 - Task 2 - PL/Perl Implementation

Pretty much the same stuff as in the Raku solution, but now I consider the function to complete succesfully only if the last node is the end.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc213.task2_plperl( int, int, int[][] )
RETURNS SETOF int
AS $CODE$
  my ( $source, $destination, $routes ) = @_;
  my @path;

  push @path, $source;
  my ( $loop ) = 1;
  my ( $current_route_index ) = 0;
  while ( $loop ) {
      my ( $current_route ) = $routes->@[ $current_route_index ];

      # skip this route if there is not a match
      $current_route_index++ and next if ( ! grep( { $_ == $path[ -1 ] } $current_route->@* ) );

      for my $node ( $current_route->@* ) {
      	  push @path, $node if ( ! grep( { $node == $_ } @path ) );

	  $loop = 0;

	  # search for the next route
	  for my $next_route_index ( ( $current_route_index + 1 ) .. scalar( $routes->@* ) ) {
	      next if ( ! grep( { $node == $_ } $routes->@[ $next_route_index ]->@* ) );
	      $current_route_index = $next_route_index;
	      $loop = 1;
	      last;
	  }

	  last if $loop;
      }

      last if $current_route_index > scalar( $routes->@* );
  }

  return undef if $path[ -1 ] != $destination;
  return [ @path ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 213 - Task 1 - PL/PgSQL Implementation

This can be solved with a single, nested, query.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc213.task1_plpgsql( a int[] )
RETURNS int[]
AS $CODE$
   WITH evens AS (
   SELECT array_agg( v ) as x
   FROM ( SELECT v FROM  unnest( a ) v
          WHERE  v % 2 = 0
          ORDER BY 1
        ) as v
   ), odds AS (
      SELECT array_agg( v ) as x
      FROM  ( SELECT v FROM  unnest( a ) v
              WHERE  v % 2 <> 0
	      ORDER BY  1 ) as v
  )
  SELECT array_cat( e.x, o.x )
  FROM evens e, odds o;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The CTE selects the odds and evens numbers out of the array, sorting them, and aggregating into a single array by means of `array_agg`. The outer query concatenates the two arrays.


<a name="task2plpgsql"></a>
## PWC 213 - Task 2 - PL/PgSQL Implementation

Cheating here: use the PL/Perl function!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc213.task2_plpgsql( s int, d int, routes int[] )
RETURNS SETOF int
AS $CODE$
   SELECT pwc213.task2_plperl( s, d, routes );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The fact is that you cannot easily manage (pass) multidimensional arrays to PL/PgSQL functions. However, I then spent some time in implementing the function, that appears as:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc213.task2_plpgsql( s int, d int, routes int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	slice_size int := 3;
	current_route_index int;
	current_route int[];
	next_route_index int;
	next_node int;
	need_loop boolean;
	node int;
	path int[];
BEGIN
	need_loop := true;
	current_route_index := 1;
<<rescan>>
	WHILE need_loop LOOP
		FOREACH node IN ARRAY routes[ current_route_index : current_route_index ] LOOP
  		         RETURN NEXT node;
			IF node = d THEN
			   EXIT;
			END IF;

			need_loop := false;
			FOR next_route_index IN current_route_index + 1 .. array_length( routes, 1 ) LOOP
			    FOREACH next_node IN ARRAY routes[ next_route_index : next_route_index ] LOOP
			    	    IF next_node = node THEN
				       current_route_index := next_route_index;
				       need_loop := true;
				       CONTINUE rescan;
				    END IF;
			    END LOOP;
		    	END LOOP;
		END LOOP;
	END LOOP;


	 IF node <> d THEN
       RAISE EXCEPTION 'Cannot find the path!';
	END IF;

	return;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The trick here is to *slice* the routes array into its own subarrays one at a time, and then check for joining of the partitions.
Since this function returns a `SETOF`, it cannot fail easily. because it is already returning results, so I decided to throw an exception thru `RAISE EXCEPTION`.
