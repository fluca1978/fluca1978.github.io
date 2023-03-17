---
layout: post
title:  "Perl Weekly Challenge 208: grep, grep and grep!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 208: grep, grep and grep!

This post presents my solutions to the [Perl Weekly Challenge 208](https://perlweeklychallenge.org/blog/perl-weekly-challenge-208/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 208 - Task 1 - Raku](#task1)
- [PWC 208 - Task 2 - Raku](#task2)
- [PWC 208 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 208 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 208 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 208 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 208 - Task 1 - Raku Implementation

Given two arrays of strings, find out matching strings within the two arrays with the minimum index sum, that means the leftmost matches.

<br/>
<br/>
```raku
sub MAIN() {
    my @first = < Perl Raku PHP Love >;
    my @second = < Raku Perl Hate >;
    my %results;

    for 0 ..^ @first.elems {
   	    %results{ $_ + @second.first( @first[ $_  ], :k ) }.push: @first[ $_ ] if ( @second.grep: @first[ $_ ] );
    }

    %results{ %results.keys.min }.join( ',' ).say;
}

```
<br/>
<br/>

I use an hash `%results` to store the sum of the indexes as key, and the string that mathes as the value. Therefore, then I only need to select the minimum key and print out the list of matching words.


<a name="task2"></a>
## PWC 208 - Task 2 - Raku Implementation

Given a list of integers, find out missing values and duplicated ones.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my %results;

    for @list.min .. @list.max {
   	    %results{ $_ } += @list.grep( $_ ).elems;
    }


    for %results.keys.sort {
	    "Duplicated value $_ (found { %results{ $_ } } times)".say if ( %results{ $_ } > 1 );
	    "Missing value $_".say if ( %results{ $_ } == 0 );
    }
}

```
<br/>
<br/>

First of all, I classify the natural sequence of numbers into the `%results` hash, couting how many times the single value appears in the givven list. Then I traverse the list and see if the value is missing or duplicated.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 208 - Task 1 - PL/Perl Implementation

Similar, even if more verbose, to the Raku implementation.
I traverse the first array, and if there are no matches, I skip over it.

If a match is found, I `grep` the indexes and take only the first one, that is the leftmost. Then, I put the sum of the indexes as the key of the `%results` hash and the string as a value.

To obtain the minimum key, I simply sort the keys and keep the first one, then I return one at a time all the strings matching with the minimum key value.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc208.task1_plperl( text[], text[] )
RETURNS SETOF text
AS $CODE$
  my ( $first, $second ) = @_;
  my %results;

  for my $a ( 0 .. $first->@* ) {
      next if ! grep( $first->[ $a ], $second->@* );

      my $b = ( grep( $first->[ $a ] eq $second->[ $_ ], 0 .. $second->@* ) )[ 0 ];
      push $results{ $a + $b }->@*, $first->[ $a ];
  }

  my $min = ( sort keys %results )[ 0 ];
  return_next( $_ ) for ( $results{ $min }->@* );

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 208 - Task 2 - PL/Perl Implementation

Similar, but again, more verbose, to the Raku solution.

First of all, I sort the values and extract the leftmost and rightmost, in order to obtain the begin and end values for a range.
Then I loop thru the range and insert into the `%results` hash the value and its counting into the original list of values.

Last, I loop again and decide which message to print out depending on the counting of the values. It is interesting to note that this function returns a *table*, that is a multicolumn object, as a Perl hash with a value and a description.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc208.task2_plperl( int[] )
RETURNS TABLE( v int, d text )
AS $CODE$
  my ( $list ) = @_;
  my %results;

  my ( $min, $max ) = ( sort $list->@* )[0, -1];
  for my $needle ( $min .. $max ) {
      $results{ $needle } += scalar grep { $_ == $needle } $list->@*;
  }

  for ( sort keys %results ) {
    next if $results{ $_ } == 1;
    return_next( { v => $_, d => "Missing value $_" } ) if ( $results{ $_ } == 0 );
    return_next( { v => $_, d => "Duplicated value $_" } ) if ( $results{ $_ } > 1 );
  }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 208 - Task 1 - PL/PgSQL Implementation

A single query result: I materialize two tables with the strings and their *ordering*, then I do a join to compute, for each string, the index sum, and last I select only those rows with the index sum equal to the minimum value found.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc208.task1_plpgsql( f text[], s text[] )
RETURNS SETOF TEXT
AS $CODE$
	WITH ta AS (
	   SELECT t, row_number() over() AS v
	   FROM unnest( f ) t
	)
	, tb AS (
	   SELECT t, row_number() over() AS v
	   FROM unnest( s ) t
	)
	, res AS (
	  SELECT ta.t, ta.v + tb.v AS v
	  FROM ta JOIN tb ON ta.t = tb.t
	)
	SELECT res.t
	FROM res
	WHERE res.v = (SELECT min( res.v ) FROM res );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 208 - Task 2 - PL/PgSQL Implementation

Another single query to do all.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc208.task2_plpgsql( l int[] )
RETURNS TABLE( v int, d text )
AS $CODE$
	WITH res AS (
	     SELECT v, count( vv ) AS c
	     FROM generate_series( l[1], l[ array_length( l, 1 ) ] ) v
	     LEFT JOIN unnest( l ) vv ON vv = v
	     GROUP BY v
	)
	SELECT v, 'Duplicated value ' || v
	FROM res
	WHERE c > 1
	UNION
	SELECT v, 'Missing value ' || v
	FROM res
	WHERE c = 0;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

I use `generate_series` to produce a list of natural numbers, and then I count how many times every number appears into the original sequence. Therefore, I am able to `UNION` all the rows with a counter equal to zero (missing) or greater than one (duplicated) and produce a table like output.
