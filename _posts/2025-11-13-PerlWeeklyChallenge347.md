---
layout: post
title:  "Perl Weekly Challenge 347: string mangling"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 347: string mangling

This post presents my solutions to the [Perl Weekly Challenge 347](https://perlweeklychallenge.org/blog/perl-weekly-challenge-347/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 347 - Task 1 - Raku](#task1)
- [PWC 347 - Task 2 - Raku](#task2)
- [PWC 347 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 347 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 347 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 347 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 347 - Task 1 in PL/Java](#task1pljava)
- [PWC 347 - Task 2 in PL/Java](#task2pljava)
- [PWC 347 - Task 1 in Python](#task1python)
- [PWC 347 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 347 - Task 1 - Raku Implementation

The first task was about to convert a date in the form `1st Dec 2025` into an ISO format `2025-12-01`:

<br/>
<br/>
```raku
sub MAIN( Str $date ) {
    my %months = 'Jan' => 1,
		 'Feb' => 2,
		 'Mar' => 3,
		 'Apr' => 4,
		 'May' => 5,
		 'Jun' => 6,
		 'Jul' => 7,
		 'Aug' => 8,
		 'Sep' => 9,
		 'Oct' => 10,
		 'Nov' => 11,
		 'Dec' => 12;

    if ( $date ~~ / ^ $<day> = ( \d{1 .. 2} ) ( 'st' | 'nd' | 'rd' | 'th' ) \s+ $<month> = ( \w+ ) \s+ $<year> = ( \d+ ) $  / ) {
		say '%04d-%02d-%02d'.sprintf: $/<year>, %months{ $/<month> }, $/<day>;
    }

}

```
<br/>
<br/>

I do a regexp named capture for all the parts and reconstruct the date in the right format.



<a name="task2"></a>
## PWC 347 - Task 2 - Raku Implementation

The second task was about rebuilding a phone string separating it into groups made from three digits and, in the case the last group is not a two or three digits, recombine it with the last-to-last group.

<br/>
<br/>
```raku
sub MAIN( Str $phone is copy ) {
    $phone .= subst( / \s+ /, '', :g );
    $phone .= subst( / <[-]> /, '', :g );
    my @groups = $phone.comb( :skip-empty ).rotor( 3, :partial );
    @groups .= map( *.join );

    if @groups[ * - 1 ].chars != 2|3 {
		my $adjust = @groups[ * - 2 ] ~ @groups[ * - 1 ];
		@groups.pop;
		@groups.pop;
		@groups.push: |$adjust.comb.rotor( 2 ).map( *.join );
    }

    @groups.join( '-' ).say;

}

```
<br/>
<br/>

First of all I remove all spaces and dashes from the `$phone` string. Then I split it into `@groups` thanks to `rotor` in groups of three elements.
If the last group has a wrong length, I build a string concatenating the last two groups, then split into two parts of two digits each.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 347 - Task 1 - PL/Perl Implementation

Same implementation as in Raku, I use named captures.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc347.task1_plperl( text )
RETURNS date
AS $CODE$

   my ( $date ) = @_;

   my %months = ( 'Jan' => 1,
		  'Feb' => 2,
		  'Mar' => 3,
		  'Apr' => 4,
		  'May' => 5,
		  'Jun' => 6,
		  'Jul' => 7,
		  'Aug' => 8,
		  'Sep' => 9,
		  'Oct' => 10,
		  'Nov' => 11,
		  'Dec' => 12 );

   if ( $date =~ / ^ (?<day>\d{1,2}) \D{2} \s+ (?<month>\D{3}) \s+ (?<year>\d{4}) $ /x ) {
      return sprintf '%04d-%02d-%02d', $+{year}, $months{ $+{month} }, $+{day};
   }
   else {
     elog( WARN, "No valid date format" );
     return undef;
   }


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 347 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, I build an array of `@groups` of digits, and in the case the last group has a wrong length, I join it with the previous group and split it again into  two digits strings:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc347.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $phone ) = @_;

   $phone =~ s/ [- ] //gx;

   my @groups;
   while ( length( $phone ) >= 3 ) {
   	 push @groups, join( '', ( ( split( //, $phone ) )[ 0 .. 2 ] ) );
	 $phone =~ s/ ^ .{3} //xg;
   }

   if ( $phone ) {
      push @groups, $phone;
   }

   if ( length( $groups[ $#groups ] ) != 2 || length( $groups[ $#groups ] ) != 3 ) {
      my ( $end, $begin ) = ( pop @groups, pop @groups );
      my $last = $begin . $end;
      push @groups, join( '', ( split( //, $last ) )[ 0 .. 1 ] );
      push @groups, join( '', ( split( //, $last ) )[ 2 .. 3 ] );
   }


   return join( '-', @groups );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 347 - Task 1 - PL/PgSQL Implementation


I use a lookup table for the months and reconstruct the date.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc347.task1_plpgsql( d text )
RETURNS date
AS $CODE$
DECLARE
	x text[];
	r text;

BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS months( i int, m text );
	TRUNCATE TABLE months;

	INSERT INTO months( i, m )
	VALUES
	  ( 1, 'Jan' )
	, ( 2, 'Feb' )
	, ( 3, 'Mar' )
	, ( 4, 'Apr' )
	, ( 5, 'May' )
	, ( 6, 'Jun' )
	, ( 7, 'Jul' )
	, ( 8, 'Aug' )
	, ( 9, 'Sep' )
	, ( 10, 'Oct' )
	, ( 11, 'Nov' )
	, ( 12, 'Dec' );

	r := '';
	x := regexp_matches( d, '(\d+).{2}\s+(\D{3})\s+(\d{4})' );

	SELECT x[ 3 ] || '-' || m.i || '-' || x[ 1 ]
	INTO r
	FROM months m
	WHERE m = x[ 2 ];


	RETURN r;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 347 - Task 2 - PL/PgSQL Implementation

Here a cheat a little: in the case the last group is of the wrong size, since I'm not so good at handling arrays into PL/PgSQL, I call the PL/Perl part.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc347.task2_plpgsql( phone text )
RETURNS text
AS $CODE$
DECLARE
	x text;
	r text;
	i int;

	groups text[];
BEGIN

	i := 0;


	FOREACH x IN ARRAY regexp_split_to_array( phone, '' ) LOOP


		IF x = '-' OR x = ' ' THEN
		   CONTINUE;
		END IF;

		RAISE INFO 'x = %', x;
		IF r IS NULL THEN
		   r := x;
		ELSE
		   r := r || x;
	        END IF;

		i := i + 1;

		IF i = 3 THEN
		   groups := groups || r;
		   r := NULL;
		   i := 0;
		END IF;
	END LOOP;

	IF i <> 0 THEN
	   groups := groups || r;
	END IF;

	IF length( groups[ array_length( groups, 1 ) ] ) THEN
	   RETURN pwc347.task2_plperl( phone );
	END IF;


	RETURN array_to_string( groups, '-' );

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 347 - Task 1 - PostgreSQL PL/Java Implementation


Here I use another trick: the day part of the string is converted into its numeric part by replacing all the suffixes, without having to check if they are present in the string.

<br/>
<br/>
```java
    @Function( schema = "pwc347",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String task1_pljava( String date ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc347.task1_pljava" );

		Map<String, Integer> months = new HashMap<String, Integer>();

		months.put( "Jan", 1 );
		months.put( "Feb", 2 );
		months.put( "Mar", 3 );
		months.put( "Apr", 4 );
		months.put( "May", 5 );
		months.put( "Jun", 6 );
		months.put( "Jul", 7 );
		months.put( "Aug", 8 );
		months.put( "Sep", 9 );
		months.put( "Oct", 10 );
		months.put( "Nov", 11 );
		months.put( "Dec", 12 );


		String[] parts = date.split( "\\s+" );
		return String.format( "%04d-%02d-%02d",
			      Integer.parseInt( parts[ 2 ] ),
			      months.get( parts[ 1 ] ),
			      Integer.parseInt( parts[ 0 ].replace( "st", "" ).replace( "nd", "" ).replace( "rd", "" ).replace( "rh", "" ) ) );
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 347 - Task 2 - PostgreSQL PL/Java Implementation

Same implementation as PL/Perl: I build groups and check the last group length to rejoin it with the previous one.

<br/>
<br/>
```java
    public static final String task2_pljava( String phone ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc347.task2_pljava" );

		phone = phone.replaceAll( "-", "" ).replaceAll( " ", "" );
		List<String> groups = new LinkedList<String>();

		String current = "";
		for( int i = 0; i < phone.length(); i++ ) {
		    current += phone.charAt( i );
		    if ( current.length() % 3 == 0 ) {
				groups.add( current );
				current = "";
		    }
		}

		if ( current.length() > 0 ) {
		    groups.add( current );
		}


		if ( groups.get( groups.size() - 1 ).length() != 2
		     || groups.get( groups.size() - 1 ).length() != 3 ) {

		    String begin = groups.get( groups.size() - 2 );
		    String end   = groups.get( groups.size() - 1 );
		    groups.remove( groups.size() - 1 );
		    groups.remove( groups.size() - 1 );

		    String last = begin + end;

		    groups.add( "" + last.charAt( 0 ) + last.charAt( 1 ) );
		    groups.add( "" + last.charAt( 2 ) + last.charAt( 3 ) );
		}

		String result = "";
		for ( String s : groups ) {
		    result += s + "-";
		}

		return result;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 347 - Task 1 - Python Implementation

Similar to PL/Java: I split the string and remove all the suffixes from the day part.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    months = { 'Jan' : 1,
               'Feb' : 2,
               'Mar' : 3,
               'Apr' : 4,
               'May' : 5,
               'Jun' : 6,
               'Jul' : 7,
               'Aug' : 8,
               'Sep' : 9,
               'Oct' : 10,
               'Nov' : 11,
               'Dec' : 12 }


    r = re.compile( r'\s+' )
    parts = r.split( args[ 0 ] )

    return '-'.join( [ parts[ 2 ],
                       str( months[ parts[ 1 ] ] ),
                       parts[ 0 ].replace( 'st', '' ).replace( 'nd', '' ).replace( 'rd', '' ).replace( 'th', '' ) ] )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 347 - Task 2 - Python Implementation

Same implementation as PL/perl.

<br/>
<br/>
```python
import sys


# task implementation
# the return value will be printed
def task_2( args ):
    phone = args[ 0 ].replace( '-', '' ).replace( ' ', '' )

    groups = []
    while len( phone ) > 0:
        if len( phone ) >= 3:
            groups.append( phone[ 0 : 3 ] )
            phone = phone[ 3: ]
        else:
            groups.append( phone )
            phone = ''

    if len( groups[ -1 ] ) != 2 and len( groups[ -1 ] ) != 3:
        begin = groups[ -2 ]
        end   = groups[ -1 ]
        last  = begin + end
        groups[ -2 ] = last[ 0 : 2 ]
        groups[ -1 ] = last[ 2 : 4 ]

    return '-'.join( groups )

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
