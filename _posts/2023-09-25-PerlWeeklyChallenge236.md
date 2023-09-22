---
layout: post
title:  "Perl Weekly Challenge 236: Arrays and Loops"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 236: Arrays and Loops

This post presents my solutions to the [Perl Weekly Challenge 236](https://perlweeklychallenge.org/blog/perl-weekly-challenge-236/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 236 - Task 1 - Raku](#task1)
- [PWC 236 - Task 2 - Raku](#task2)
- [PWC 236 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 236 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 236 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 236 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 236 - Task 1 - Raku Implementation

The first task was about having to provide a remainder after selling an item that costs `$5`. Only `$10`, `$5` and `$20` notes are supported. The program must return a true of false depending if it can sell an item to every one offering an input note.

<br/>
<br/>
```raku
sub MAIN( *@cash where { @cash.grep( * % 5 == 0 ).elems == @cash.elems } ) {

    my %remainder;
    %remainder{ $_ } = 0 for 5,10,20;

    for @cash -> $current_cash {
		%remainder{ $current_cash }++;
		next if $current_cash == 5;


		if ( $current_cash == 10 and %remainder{ 5 } > 0 ) {
		    %remainder{ 5 }--;
		}
		elsif $current_cash == 20  {
		    if %remainder{ 10 } > 0 && %remainder{ 5 } > 0 {
				%remainder{ 5 }--;
				%remainder{ 10 }--;
		    }
		    elsif %remainder{ 10 } == 0 && %remainder{ 5 } > 3 {
				%remainder{ 5 } -= 3;
		    }
		    else {
				'False'.say and exit;
		    }
		}
		else {
		    # cannot proceed
		    'False'.say and exit;
		}

    }

    'True'.say;
}

```
<br/>
<br/>

The idea is this: `%remainder` stores the quantity of notes for every value that the seller receives.
Since the item costs `$5`, it does not have to provide a remainder if the buyer payes exactly five bucks. Otherwise, if the buyer pays ten buck, a `$5` must be returned. The `$20` case is a little more complex, since the remained could be `$5 + $10` or `$5 + $5 + $5`.
In any case, as soon as the seller cannot provide the remainder, the program ends.


<a name="task2"></a>
## PWC 236 - Task 2 - Raku Implementation

This was about finding loops in a given array: start at an index, get a value, move to the index specified in the value, and continue. If you come back to the starting position, there is a loop.

<br/>
<br/>
```raku
sub MAIN( *@nums  where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {

    my @loops;
    for 0 ..^ @nums.elems -> $current-start {
	my @current-path = ();

	@current-path.push: @nums[ $current-start ];

	my $next = @nums[ $current-start ];
	while ( 0 < $next < @nums.elems ) {
	    @current-path.push: @nums[ $next ];

	    if @nums[ $next ] == $current-start {
				# loop detected
				@loops.push: @current-path;
				last;
	    }

	    $next = @nums[ $next ];
	}

    }

    @loops.elems.say;
}

```
<br/>
<br/>

The idea is to traverse the array with the `$next` index, and if the indexed value is the same as the starting one, a loop has been detetcted and added to the `@loops` array, so that I can count the elements.

This program provides more loops than the example tells, and apparently they are correct.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 236 - Task 1 - PL/Perl Implementation

Same as in Raku, but this time I `return` a false value immediatly.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc236.task1_plperl( int[] )
RETURNS boolean
AS $CODE$
   my ( $cash ) = @_;

   my $remainder = {};

   for my $current_cash ( $cash->@* ) {
       $remainder->{ $current_cash }++;
       next if $current_cash == 5;

       if ( $current_cash == 10 ) {
       	  return 0 if ( $remainder->{ 5 } == 0 );
		  $remainder->{ 5 }--;
       }

       if ( $current_cash == 20 ) {
          if ( $remainder->{ 10 } > 0 && $remainder->{ 5 } > 0 ) {
			  $remainder->{ 5 }--;
			  $remainder->{ 10 }--;
	  	  }
	  	  elsif ( $remainder->{ 10 } == 0 && $remainder->{ 5 } >= 3 ) {
	  	     $remainder->{ 5 } -= 3;
	  	  }
	  	  else {
	  	    return 0;
	  	  }


       }

   }

   return 1;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 236 - Task 2 - PL/Perl Implementation

Same as the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc236.task2_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $nums ) = @_;
   my @loops;

   for my $current_start ( 0 .. $nums->@* ) {
       my @current_path = ();

       push @current_path, $nums->[ $current_start ];
       my $next = $nums->[ $current_start ];

       while ( 0 < $next < scalar( $nums->@* ) ) {
       	     push @current_path, $nums->[ $next ];

	     if ( $nums->[ $next ] == $current_start ) {
	     	push @loops, \@current_path;
		last;
	     }

	     $next = $nums->[ $next ];
       }
   }


   return scalar @loops;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 236 - Task 1 - PL/PgSQL Implementation

This time I use a temporary table as an hash to keep track of how many notes I have with a specific value.
This makes the code a little more verbose, because I have to update the table every time I encounter a note.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc236.task1_plpgsql( cashing int[] )
RETURNS boolean
AS $CODE$
DECLARE
	current_cash int;
	available_qty_10 int;
	available_qty_5 int;
BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS remainder( cash int, quantity int default 1 );
	TRUNCATE TABLE remainder;
	INSERT INTO remainder( cash, quantity )
	VALUES
	(5, 0 ), ( 10, 0 ), ( 15, 0 );

	FOREACH current_cash IN ARRAY cashing LOOP
		UPDATE remainder
		SET quantity = quantity + 1
		WHERE cash = current_cash;

		IF current_cash = 5 THEN
		   CONTINUE;
		END IF;

		IF current_cash = 10 THEN
		   SELECT quantity
		   INTO available_qty_5
		   FROM remainder
		   WHERE cash = 5;

		   IF available_qty_5 > 0 THEN
		      UPDATE remainder
		      SET quantity = quantity - 1
		      WHERE cash = 5;
		   ELSE
		      RETURN false;
		   END IF;
		END IF;


		IF current_cash = 20 THEN
		   SELECT quantity
		   INTO available_qty_10
		   FROM remainder
		   WHERE cash = 10;

		   SELECT quantity
		   INTO available_qty_5
		   FROM remainder
		   WHERE cash = 5;

		   IF available_qty_10 > 0 AND available_qty_5 > 0 THEN
		      UPDATE remainder
		      SET quantity = quantity - 1
		      WHERE cash IN ( 5, 10 );
		   ELSIF available_qty_10 = 0 and available_qty_5 >= 3 THEN
		      UPDATE remainder
		      SET quantity = quantity - 3
		      WHERE cash = 5;
		   ELSE
		     RETURN false;
		   END IF;
		END IF;
	END LOOP;

	return true;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 236 - Task 2 - PL/PgSQL Implementation

This is a shorter solution with respect to the Raku and PL/Perl ones, but it is mainly due to the fact that I don't keep track of the paths that lead to a loop. The idea behind the solution is the same as the other implementations.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc236.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	current_start int;
	loops int := 0;
	next int;
BEGIN

	FOR current_start in 1 .. array_length( nums, 1 ) LOOP
		next := nums[ current_start ];

		WHILE next >= 1 AND next <= array_length( nums, 1 ) LOOP
		      IF nums[ next ] = current_start THEN
		      	 loops := loops + 1;
			 EXIT;
		      END IF;

		      next := nums[ next ];
		END LOOP;
	END LOOP;

	RETURN loops;
END

$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
