---
layout: post
title:  "Perl Weekly Challenge 175: Sunday Math!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 175: Sunday Math!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 175](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0175/){:target="_blank"}.

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
## PWC 175 - Task 1

Find out the last sundays in all the months in a given year:



<br/>
<br/>
```raku
sub MAIN( Int $year = 2022 ) {

    my @dates;

    for 1 .. 12 {
        my $day = Date.new( year => $year,
                            month => $_ ) # day automatically set to 1
                      .last-date-in-month;

        $day .= pred  while ( $day.day-of-week != 7 );
        @dates.push: $day;
    }

    @dates.join( "\n" ).say;
}

 ```
<br/>
<br/>


The idea is simple: for every month I create a `Date` object with the last day of the month, then I move backward to find out the first sunday, and that is the last sunday in the month.

<a name="task2"></a>
## PWC 175 - Task 2

Gosh, a too much complex, according to me, alghoritm to find out *perfect totient numbers*. I split the work into two parts:

<br/>
<br/>
```raku
use Prime::Factor;

sub MAIN( Int $limit where { $limit > 0 } = 20 ) {
     my @totients = lazy (0 .. *).map: { $_ *  [*] $_.&prime-factors.squish.map: 1 - 1/*  };
     my @perfect-totients = (3, * + 2 ... *).grep: -> $current {
         $current ==  [+] @totients[ $current ] , { @totients[ $_ ] }  ... 1
     };

     @perfect-totients[ 0 .. $limit ].join( ', ' ).say;
}

```
<br/>
<br/>

The `@totients` is a lazy array that computes the totient numbers. Here it is important to use `squish`, to avoid too much duplicates in the sequence. In the first implementation I forgot to use it, and the result was not working at all!
<br/>
The `@perfect-totients` array iterates on a sequence that moves forward of two unitys at a time, and checks if the sum of the totient numbers is the same as the number itself, in such case the number is perfect.

<a name="task1plperl"></a>
## PWC 175 - Task 1 in PostgreSQL PL/Perl

Very likely the Raku solution:

<br/>
<br/>

``` sql
CREATE SCHEMA IF NOT EXISTS pwc175;

CREATE OR REPLACE FUNCTION
pwc175.task1_plperl( int )
RETURNS SETOF DATE
AS $CODE$

use DateTime;
my ( $year ) = @_;

for ( 1 .. 12 ) {
    my $day = DateTime->last_day_of_month( year => $year, month => $_ );
    $day->add( days => -1 ) while( $day->dow != 7 );
    return_next( $day->ymd );
    ;
}

return undef;
$CODE$
LANGUAGE plperlu;
```
<br/>
<br/>

I used `plperlu` because I needed to load the `DateTime` module.


<a name="task2plperl"></a>
## PWC 175 - Task 2 in PostgreSQL PL/Perl

The math done in Perl. This time I used anonymous subroutines and a module to do recursive calls:


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc175.task2_plperl( int )
RETURNS SETOF INT
AS $CODE$

use ntheory qw/euler_phi/;
use Sub::Recursive;

my ( $limit ) = @_;


my $totients = recursive {
   my ( $t ) = @_;
   return euler_phi( $t )
               + ( $t == 2
                      ? 0
                      : $REC->( euler_phi( $t ) ) );
};

for ( 2 .. 99999 ) {
    return_next( $_ ) and $limit-- if ( $_ == $totients->( $_ ) );
    last if $limit <= 0;
}

return undef;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

Luckily, the `ntheory` module provides an `euler_phi` function that does the same job as the `@totients` block of code in the Raku implementation.


<a name="task1plpgsql"></a>
## PWC 175 - Task 1 in PostgreSQL PL/PgSQL

Similar approach as in Raku: generate all days within an year, and then search for sundays and keep only the last one in a given month:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc175.task1_plpgsql( year int DEFAULT 2022 )
RETURNS SETOF DATE
AS $CODE$
DECLARE
        last_sunday date;
        d           date;

BEGIN

        FOR d IN SELECT sunday FROM
                               generate_series( to_date( year || '-01-01' ),
                                                to_date( year || '-12-31' ),
                                                '1 day'::interval ) sunday
                                               WHERE
                                               extract( dow from sunday ) = 0
                                               ORDER BY 1 ASC
                                               LOOP
           IF last_sunday IS NULL THEN
              last_sunday := d;
           END IF;

           IF extract( day from last_sunday ) < extract( day from d ) AND extract( month from last_sunday ) = extract( month from d ) THEN
            last_sunday := d;
            CONTINUE;
          END IF;

           IF extract( month from last_sunday ) <> extract( month from d ) THEN
              RETURN NEXT last_sunday;
              last_sunday := d;
           END IF;

       END LOOP;

       RETURN NEXT last_sunday;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

<a name="task2plpgsql"></a>
## PWC 175 - Task 2 in PostgreSQL PL/PgSQL

Too much complicated to implement in pure PL/PgSQL, so I decided to delegate to the Perl function:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc175.task2_plpgsql( l int DEFAULT 20)
RETURNS SETOF INT
AS $CODE$
BEGIN
        RETURN QUERY SELECT pwc175.task2_plperl( l );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
