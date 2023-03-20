---
layout: post
title:  "Perl Weekly Challenge 209: grep and loop"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 209: grep and loop

This post presents my solutions to the [Perl Weekly Challenge 209](https://perlweeklychallenge.org/blog/perl-weekly-challenge-209/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 209 - Task 1 - Raku](#task1)
- [PWC 209 - Task 2 - Raku](#task2)
- [PWC 209 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 209 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 209 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 209 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 209 - Task 1 - Raku Implementation

The first task was about finding out if a sequence of binary digits ends with a single `0` or not. The task was appearing much more complex, providing a translation from two digits into a letter, and seeking for an ending letter. The Raku implementation remaps the binary string into a sequence of letters, but as you will see in the PL/Perl implementations, it could have been simpler.

<br/>
<br/>
```raku
sub MAIN( *@bits where { @bits.grep( * ~~ any(1, 0) ).elems == @bits.elems } ) {
    my @chars = @bits.rotor( 2, :partial ).map(
	                                        {
						       given ( $_.join ) {
						           when '10' { 'b' }
						           when '11' { 'c' }
						           when '0'  { 'a' }
						           default   { 'z' }
						       }
                                                 } );

    '1'.say and exit if ( @chars[ * - 1 ] ~~ 'a' );
    '0'.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 209 - Task 2 - Raku Implementation

The second task was about merging accounts where the same email appeared. It was not as simple as it could sound due to the lack of structure of the accounts themselves: everything is represented as a list.

<br/>
<br/>
```raku
sub MAIN() {

    my  @accounts =
	( "A", "a1@a.com", "a2@a.com" ),
        ( "B", "b1@b.com" ),
        ( "A", "a3@a.com", "a1@a.com" );

    my @merge;

    for 0 ..^ @accounts.elems {
	next if ! @accounts[ $_ ];
	my @current-emails = @accounts[ $_ ][ 1 .. * ];
	for $_ ^..^ @accounts.elems {
	    next if ! @accounts[ $_ ];
	    my @emails = @accounts[ $_ ][ 1 .. * ];
	    my @match  = @current-emails.grep( $_ )  for @emails;
	    if ( @match ) {
				@merge.push: [ @accounts[ $_ ][ 0 ], unique( sort(  |@emails, |@current-emails ) ) ];
				@accounts[ $_ ] = Nil;
				last;
	    }
	    else {
	            @merge.push: [ @accounts[ $_ ] ];
	    }
	}
    }

    @merge.join( "\n" ).say;
}

```
<br/>
<br/>

The idea is to iterate other the `@accounts` and extract the `@current-emails` list, then proceed to another account and count how many emails are overlapped by means of `grep`. If there are a few `@match`es, then the two accounts have to be merged into the `@merge` array and the current account is deleted from the source array, otherwise the single account is added with no merging at all.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 209 - Task 1 - PL/Perl Implementation

Simpler than the Raku implementation: if the string has an odd length and ends with zero, there is a success. Otherwise, if the string has an even length, it must terminate with at least two zeros.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc209.task1_plperl( text )
RETURNS int
AS $CODE$
  my ( $string ) = @_;
  my @bits = split( '', $string );
  return 1 if ( @bits % 2 != 0 && @bits[ - 1 ] == 0 );
  return 1 if ( @bits % 2 == 0 && @bits[ - 1 ] == 0 && @bits[ - 2 ] == 0 );
  return 0;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 209 - Task 2 - PL/Perl Implementation

Quite complex solution: I did a two step query to get the results.

First of all, all accounts are stored into a table. Then I query the table counting which emails have duplicates. With those emails, I build a new query that gives me all the entries of accounts related to such duplicated emails, and I merge them creating a table entry that is returned to the caller.

<br/>
<br/>
```perl
DROP TABLE IF EXISTS pwc209.accounts;
CREATE TABLE IF NOT EXISTS pwc209.accounts ( a_name text, a_email text );
TRUNCATE TABLE pwc209.accounts;
INSERT INTO pwc209.accounts
VALUES ( 'A', 'a1@a.com' )
, ('A', 'a2@a.com' )
, ( 'B', 'b@b.com'  )
, ( 'A', 'a3@a.com' )
, ( 'A', 'a1@a.com'  );


CREATE OR REPLACE FUNCTION
pwc209.task2_plperl()
RETURNS TABLE( a text, e text[] )
AS $CODE$

   my $result_set = spi_exec_query( " select a_email, count(*) from pwc209.accounts group by a_email having count(*) > 1 " );



   my @duplicated_emails;
   for  ( 0 .. $result_set->{ processed } - 1 ) {
      my $row = $result_set->{ rows }[ $_ ];
      push @duplicated_emails,  $row->{ a_email };
   }



   my $query = sprintf qq/ with accs AS ( select distinct a_name from pwc209.accounts where a_email IN (%s) )
select a.a_name, a_email  from pwc209.accounts a, accs where a.a_name = accs.a_name /
, join( ',',  map( { "'$_'"  } @duplicated_emails ) );
   $result_set = spi_exec_query( $query );

   my $to_return = {};
    for  ( 0 .. $result_set->{ processed } - 1 ) {
      my $row = $result_set->{ rows }[ $_ ];
      return_next( $to_return ) if ( $to_return->{ a } && $to_return->{ a } ne $row->{ a_name } );

      $to_return->{ a } = $row->{ a_name };
      next if ( grep { $_ eq $row->{ a_email } } $to_return->{ e }->@* );
      push $to_return->{ e }->@*, $row->{ a_email };
    }

   return_next( $to_return );
return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 209 - Task 1 - PL/PgSQL Implementation

Same idea as the PL/Perl implementation, using a regular expression to handle the string ending zeros.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc209.task1_plpgsql( b text )
RETURNS int
AS $CODE$
DECLARE
BEGIN
	IF ( length( b ) % 2 = 0 AND b ~ '00$' ) OR ( length( b ) % 2 = 1 AND b ~ '0$' ) THEN
	   RETURN 1;
	ELSE
	  RETURN 0;
	END IF;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 209 - Task 2 - PL/PgSQL Implementation

Using the same table data structure, a single query with a CTE can solve the problem:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc209.task2_plpgsql()
RETURNS TABLE( a text, e text )
AS $CODE$

WITH duplicated_emails AS ( SELECT a_email FROM pwc209.accounts GROUP BY a_email HAVING COUNT(*) > 1 )
, duplicated_accounts AS ( SELECT a_name FROM pwc209.accounts WHERE a_email IN ( SELECT a_email FROM duplicated_emails ) )
SELECT distinct( a_name, a_email )
FROM pwc209.accounts WHERE a_name IN ( SELECT a_name FROM duplicated_accounts );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The `duplicaed_emails` reports all the emails that are duplicated; the results are then pulled into `duplicated_accounts` to get out all the duplicated accounts. Finally, a `distinct` query reports all the accounts as merged.
