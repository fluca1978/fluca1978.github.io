---
layout: post
title:  "Perl Weekly Challenge 259: too much work to do!"
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

# Perl Weekly Challenge 259: too much work to do!

This post presents my solutions to the [Perl Weekly Challenge 259](https://perlweeklychallenge.org/blog/perl-weekly-challenge-259/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 259 - Task 1 - Raku](#task1)
- [PWC 259 - Task 2 - Raku](#task2)
- [PWC 259 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 259 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 259 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 259 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 259 - Task 1 in PL/Java](#task1pljava)
- [PWC 259 - Task 2 in PL/Java](#task2pljava)
- [PWC 259 - Task 1 in Python](#task1python)
- [PWC 259 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 259 - Task 1 - Raku Implementation

The first task was about computing a destination date given an initial date, an offset (in days) and a set of dates to skip.
THe script has to compute the final date skipping weekends and all the days into the optionally specified list of holidays.

<br/>
<br/>
```raku
sub MAIN( $date,
	  Int $offset is copy,
	  *@holidays
				where { $date ~~ / ^ \d ** 4 <[-]>\d ** 2 <[-]> \d ** 2 $ / and $offset > 0 } ) {

    my $day = DateTime.new( $date, formatter => { sprintf "%04d-%02d-%02d", .year, .month, .day } );


    while ( $offset > 0 ) {
		$day .= later( days => 1 );

		# skip weekends
		while ( $day.day-of-week == any( 7, 6 ) ) {
		    $day .= later( days => 1 );
		}

		while ( @holidays.elems > 0 && @holidays.grep( * ~~ $day.Str ) ) {
		    $day .= later( days => 1 );
		}


		$offset--;
    }

    $day.say;

}

```
<br/>
<br/>

The idea is to store the `DateTime` object into a `$day` variable that is increased by one day at a time (by means of `later`) looping for every `$offset` days specified. The `$day` is moved forward one day at a time if it is a weekend or is found in the holidays array.

<a name="task2"></a>
## PWC 259 - Task 2 - Raku Implementation

The task asked to write a parser for a line of text in the form of `{% id param=value param=value %}`.
I have not implemented this task in a very accurate way, since it is too much regular expression work for me!
<br/>
<br/>
```raku
sub MAIN() {

    my %parsed;

    my $line = '{%  youtube video=foobar password=xyz abc=def donald="duck here \"escaped\" " %}';

    my regex id { \w+ };
    my regex option {  $<name>= [ \w+ ] \s* <[=]> $<value>= [ \w+ | \" \w+ \s* .* \" ] };
    my regex opening { <[{]> <[%]> };
    my regex closing { <[%]> <[}]> };

    if ( $line ~~ / ^ <opening> \s+ <id> \s+ <option>* % \s  \s* <closing> $ / ) {

		%parsed<name> = $/<id>;

		say $/<option>;
		for $/<option> {
		    my ( $key, $value ) = .split( '=' );
		    %parsed<fields>{ $key } = $value;
		}
    }

    say %parsed;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 259 - Task 1 - PL/Perl Implementation

Similar implementation to the Raku one, with `DateTime` object used for days.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc259.task1_plperl( date, int, date[] )
RETURNS date
AS $CODE$

use DateTime;

   my ( $when, $offset, $holidays ) = @_;

   $when =~ / ^ (?<year>\d{4}) [-] (?<month>\d{2}) [-] (?<day>\d{2}) $ /x;

   my $day = DateTime->new( year => $+{ year}, month => $+{ month }, day => $+{ day }	 );

   while ( $offset > 0 ) {
   	 $day->add( days => 1 );
	 $offset--;

	 # skip weekends
	 while ( $day->day_of_week == 6 || $day->day_of_week == 7 ) {
	       $day->add( days => 1 );
	 }

	 if ( $holidays->@* ) {
	    while( grep { $_ eq $day->ymd } $holidays->@* ) {
	       $day->add( days => 1 );
	    }
	 }
   }

   return $day->ymd;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 259 - Task 2 - PL/Perl Implementation


Again, a disaster of regular expressions!

This time I tried to implement a kind of staging parser, where each character of the paramaters string is analyzed to understand if accumulating a key or a value for a pair.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc259.task2_plperl( text)
RETURNS TABLE( id text, field_name text, field_value text )
AS $CODE$

   my ( $line ) = @_;

   if ( $line =~ / ^ [{] [%] \s* (?<id>\w+) \s* (?<options>.*) \s* [%] [}] $ /x ) {
      my $id = $+{ id };
      my ( $name, $value ) = ( '', '' );
	if ( $+{ options } ) {
	   my $is_value = 0;
	   my $allowed_spaces = 0;
	   my $previous = '';

	   for ( split //, $+{ options } ) {


	       $is_value = 1 and $previous = $_ and next if ( $_ eq '=' );
	       $allowed_spaces = 1 and $previous = $_ and next if ( $_ eq '"' and $previous eq '=' );

	       $name .= $_ if ( ! $is_value );
	       $value .= $_ if ( $is_value );

	       if ( $is_value
	           && ( ( $_ eq ' ' && ! $allowed_spaces )
		     || ( $_ eq '"' && $previous ne '\\' && $allowed_spaces ) )
		     ) {
	       	  # stop here!
		  $value =~ s/^\s*|\s*$//g;
		  $value =~ s/^["]|["]$//g;
		  $value =~ s/\\"/"/g;
     		  return_next( { id => $id ,
      		     field_name => $name,
		     field_value => $value } );

		  ( $name, $value, $is_value, $allowed_spaces ) = ( '', '', 0, 0 );
	       }

	       $previous = $_;
	   }


	   }
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 259 - Task 1 - PL/PgSQL Implementation

Quite simple to implement, using the same approach of PL/Perl:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc259.task1_plpgsql( day date, how_many_days int, holidays date[] )
RETURNS date
AS $CODE$
DECLARE
	current_holiday date;
BEGIN

	WHILE how_many_days > 0 LOOP
	      day := day + 1;

	      WHILE extract( dow from day ) IN ( 0, 6 ) LOOP
	      	 day := day + 1;
              END LOOP;

	      FOREACH current_holiday IN ARRAY holidays LOOP
	      	      IF current_holiday = day THEN
		      	 day := day + 1;
		      END IF;
	      END LOOP;

	      how_many_days := how_many_days - 1;
	END LOOP;

	RETURN day;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 259 - Task 2 - PL/PgSQL Implementation

Here I faked!
I used the PL/Perl (not complete) implementation, since there is too much work to do in PL/PgSQL with this regular expressions.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc259.task2_plpgsql( line text )
RETURNS TABLE( id text, field_name text, field_value text )
AS $CODE$
   SELECT pwc259.task2_plperl( line );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 259 - Task 1 - PostgreSQL PL/Java Implementation

Same implementation as PL/Perl and the others, with the care of using `java.sql.Date` objects.


<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc259",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final java.sql.Date task1_pljava( Date startDay, int how_many, Date[] holidays ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc259.task1_pljava" );

		Calendar day = Calendar.getInstance();
		day.setTime( startDay );

		while ( how_many > 0 ) {
		    day.add( Calendar.DAY_OF_YEAR, 1 );

		    while ( day.get( Calendar.DAY_OF_MONTH ) == Calendar.SUNDAY
			    || day.get( Calendar.DAY_OF_MONTH ) == Calendar.SATURDAY )
			day.add( Calendar.DAY_OF_YEAR, 1 );

		    if ( holidays != null )
			for ( Date skip : holidays )
			    if ( skip.equals( day.getTime() ) )
				day.add( Calendar.DAY_OF_YEAR, 1 );

		    how_many--;
		}

		return new java.sql.Date( day.getTimeInMillis() );
    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 259 - Task 2 - PostgreSQL PL/Java Implementation

The implementation of this task has been done partially, using nested regular expressions.


<br/>
<br/>
```java
public class Task2 implements ResultSetProvider {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc259",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final ResultSetProvider task2_pljava( String line ) throws SQLException {
		return new Task2( line );
    }


    public Task2( String line ) throws SQLException {
		params = new LinkedList< List<String> >();
		parse( line );
    }


    private final void parse( String line ) throws SQLException {
		Pattern pattern = Pattern.compile( "[{][%] (\\w+)\\s*(.*)\\s* [%][}]" );
		Matcher matcher = pattern.matcher( line );

		if ( matcher.find() ) {
		    id = matcher.group( 1 );

		    Pattern subPattern = Pattern.compile( "(\\w+)[=](\\w+)\\s*" );
		    Matcher subMatch   = subPattern.matcher( matcher.group( 2 ) );

		    while ( subMatch.find() ) {
			List<String> current = new LinkedList<String>();
			current.add( subMatch.group( 1 ) );
			current.add( subMatch.group( 2 ) );
			params.add( current );

		    }
		}
		else
		    throw new SQLException( "Cannot parse " + line );

    }

    private List< List<String> > params;
    private String id;

    @Override
    public boolean assignRowValues(ResultSet rs, int row)
	throws SQLException {

		// stop the result set
		if ( row >= params.size() )
		    return false;

		rs.updateInt( 1, row );
		rs.updateString( 2, id );
		rs.updateString( 3, params.get( row ).get( 0 ) );
		rs.updateString( 4, params.get( row ).get( 1 ) );
		return true;
    }

    @Override
    public void close() {
    }
}

```
<br/>
<br/>

The idea is to perform the parsing within the `parse()` method, that will store the parameters into the `params` list, so that the function can return such list as a result set.

# Python Implementations

<a name="task1python"></a>
## PWC 259 - Task 1 - Python Implementation

Same implementation as in other tasks, but requires a `timedelta` to add one day at a time!

<br/>
<br/>
```python
import sys
from datetime import date, timedelta

# task implementation
# the return value will be printed
def task_1( args ):
    day      = date.fromisoformat( args[ 0 ] )
    offset   = int( args[ 1 ] )
    holidays = list( map( lambda x: date.fromisoformat( x ), args[ 2: ] ) )
    one_day  = timedelta( days=1 )

    while offset > 0 :
        day += one_day

        while day.weekday() >= 6 or day.weekday() == 0:
            day += one_day

        while day in holidays :
            day += one_day

        offset -= 1

    return day


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 259 - Task 2 - Python Implementation

A character-at-a-time parser, not a very good implementation, sob!


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    line = args[ 0 ]

    parsed = {}
    id = ''
    key = ''
    value = ''
    is_value = False
    is_key  = False

    for c in line:
        if c == '{' :
            continue
        elif c == '%':
            continue
        elif c == '}':
            return parsed
        elif c == ' ':
            if not 'id' in parsed and len( id ) > 0 :
                parsed[ 'id' ] = id
                parsed[ 'fields' ] = []
                is_key = True
                continue
            elif 'fields' in parsed:
                parsed[ 'fields' ].append( { key : value } )
                is_key = True
                key = ''
                value = ''
                is_value = False

        elif c != '=' :
            if not 'id' in parsed:
                id += c
                is_value = False
                is_key = False
                continue
            else:
                if is_key:
                    key += c
                else:
                    value += c

        elif c == '=':
            if is_key:
                is_value = True
                is_key   = False




    return parsed


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
