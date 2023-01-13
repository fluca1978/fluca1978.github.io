---
layout: post
title:  "Perl Weekly Challenge 183: arrays and days"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 183: arrays and days

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 183](https://perlweeklychallenge.org/blog/perl-weekly-challenge-183/){:target="_blank"}.

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
## PWC 183 - Task 1

Piece of cake!
<br/>
Given a list of array references, discard those that are the same array. The good thing is that the /smart match/ operator will do the job for us. So I keep a `@clear` list and do a nested loop to see if a given array is the same as another one, and if it is I keep looping, otherwise I push the found array reference into my clear list.


<br/>
<br/>
```raku
sub MAIN() {
    my @list = [1,2], [3,4], [5,6], [1,2];
    my @clear;

    for 0 ..^ @list.elems -> $left {
        my $found = False;
        for $left ^..^ @list.elems -> $right {
            $found = True and last if @list[ $left ] ~~ @list[ $right ];
        }

        next if $found;
        @clear.push: @list[ $left ] if ! $found;
    }

    @clear.join( ',' ).gist.say;
}

 ```
<br/>
<br/>



<a name="task2"></a>
## PWC 183 - Task 2
Given two /YYYY-MM-DD/ dates, compute the difference in years and days.
<br/>
The good news here is that `Date` provides a `daycount` emthod that coutns the days from the epoch, so it just suffice to get the difference in days, and round it up to years and the remaining as days.

<br/>
<br/>
```raku
sub MAIN( Str $begin-date where { / \d ** 4 '-' \d ** 2 '-' \d ** 2 / }
          , Str $end-date where { / \d ** 4 '-' \d ** 2 '-' \d ** 2 / } ) {

    my @dates = Date.new( $begin-date )
                , Date.new( $end-date );

    my $days = abs( @dates[ 0 ].daycount - @dates[ 1 ].daycount );
    my $years = $days >= 365 ?? ($days / 365).Int !! 0;
    $days = $days >= 365 ?? $days % 365 !! $days;

    "$begin-date - $end-date = $years years and $days days".say;
}

```
<br/>
<br/>


<a name="task1plperl"></a>
## PWC 183 - Task 1 in PostgreSQL PL/Perl

Here there's some more work to see if arrays are the same in depth, but the overall approach is the same as in the Raku solution.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc183.task1_plperl()
RETURNS int[]
AS $CODE$

my @list = ( [1,2], [3,4], [5,6], [1,2] );
my @clear;


for my $index1 ( 0 .. $#list - 1 ) {
    my $array_found = 0;
    for my $index2 ( $index1 + 1 .. $#list ) {
        my $left  = $list[ $index1 ];
        my $right = $list[ $index2 ];

        my $found = 0;
        for my $item_left ( @$left ) {
            for my $item_right ( @$right ) {
                $found++ and last if $item_left == $item_right;
            }
        }

        $array_found++ if ( $found == scalar( @$left ) );
    }

    push @clear, $list[ $index1 ] if ( ! $array_found );
}

return \@clear;

$CODE$
LANGUAGE plperl;
```
<br/>
<br/>

Note that I need to return an array reference for the PostgreSQL-Perl machinery to convert a Perl array into a PostgreSQL array.




<a name="task2plperl"></a>
## PWC 183 - Task 2 in PostgreSQL PL/Perl

Similarly to the Raku approach, I use `DateTime` to get the number of days of difference between two dates and then round up as years and remaining days. The result is spurt as an array, where the first value is the number of years, and the second value is the number of days.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc183.task2_plperl( text, text )
RETURNS int[]
AS $CODE$

use DateTime;
my @dates;

for my $current_date ( @_ ) {
    $current_date =~ /^(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})$/;
    push @dates, DateTime->new( year  => $+{ year },
                                month => $+{ month },
                                day   => $+{ day } );
}

my $difference = $dates[ 0 ]->subtract_datetime( $dates[ 1 ] );
return [ $difference->in_units( qw/ years days / ) ];

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 183 - Task 1 in PostgreSQL PL/PgSQL

Uhm, PostgreSQL does not have *array references*, so I simply invoke the PL/Perl function as a placeholder for the solution.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc183.task1_plpgsql()
RETURNS int[]
AS $CODE$
   SELECT pwc183.task1_plperl();
$CODE$
LANGUAGE sql;
```
<br/>
<br/>


<a name="task2plpgsql"></a>
## PWC 183 - Task 2 in PostgreSQL PL/PgSQL

Here it does suffice to do a simple subtraction to get the days as a date difference, and then I can simply return an array where the first element is the number of years, and the second is the number of remaining days.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc183.task2_plpgsql( d1 date, d2 date )
RETURNS int[]
AS $CODE$
DECLARE
        days  int;
        years int;
BEGIN
        days := abs( d2 - d1 );

        IF days >= 365 THEN
           years := days / 365;
           days  := days % 365;
        ELSE
           years := 0;
        END IF;

        RETURN ARRAY[ years, days ]::int[];
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>
