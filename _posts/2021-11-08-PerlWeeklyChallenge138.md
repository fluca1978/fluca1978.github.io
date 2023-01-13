---
layout: post
title:  "Perl Weekly Challenge 138: split working days"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 138: split working days

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 138](https://perlweeklychallenge.org/blog/perl-weekly-challenge-138/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg)





<a name="task1"></a>
## PWC 138 - Task 1

The first task was about computing how many *working days* there are in a given year. A working day is a day that is not either a Sunday or a Saturday.
<br/>
I decided to implement it as huge loop:

<br/>
<br/>
```raku
sub MAIN( Int $year where { $year ~~ / \d ** 4 / } && $year > 1900,
        Bool :$verbose = False ) {
    my Date $date .= new: year => $year, day => 1, month => 1;
    my Date $stop .= new: year => $year, day => 31, month => 12;
    my $work-days = 0;
    while ( $date <= $stop ) {
        $work-days += 1 if $date.day-of-week != any( 6, 7 );
        $date = $date + 1;
    }

    "$year has $work-days work days".say if $verbose;
    $work-days.say if ! $verbose;
}
``` 

<br/>
<br/>

The idea is that `$date` contains the current date, starting from the very first day of the year. I don't take into account holidays. The `$stop` represents the last day of the year, so my idea is to iterate one day at time from `$start` to `$stop`.
If `$date` represents a weekly day that is not either `6` or `7` (the Raku `Date` representation of Saturday and Sunday), I increment the `$work-days` counter. Note that I use a *junction* to quickly test a double expressed condition.




<a name="task2"></a>
## PWC 138 - Task 2

The second task was assigning a perfect square number, and the program has to find out if the square of the number can be split as a combination of the digits of the number itself.
<br/>
My solution is not ideal, but should work in many cases.

<br/>
<br/>
```raku
sub MAIN( Int:D $n where { $n > 0 } ) {
    my $qrt = $n.sqrt;
    my @digits = $n.split( '', :skip-empty );

    # short circuit: esay case
    1.say and exit if @digits.sum == $qrt;

    # try to aggregate from left to right
    for 1 ..^ @digits.elems {
        last if @digits[ 0 .. $_ ].join.Int > $qrt;
        '1'.say and exit if @digits[ 0 .. $_ ].join + @digits[ $_ + 1 .. * - 1 ].sum == $qrt;
    }
    
}


```
<br/>
<br/>

I compute the `sqrt` of the given input number, that I assign to the `$qrt` variable (there is no `s` because the `$` sigil appears as an `s`). I then extract every `@digits` and look if the sum of single digits provides the square root, and this case I can stop the program.
<br/>
Otherwise, I try to aggregate the digits from left to right **only** (thus, this is not ideal) and see if the sum of the slices of the `@digits` array provides me the searched for result.




<a name="task1pg"></a>
## PWC 138 - Task 1 in PostgreSQL `plpgsql`

The implementation in `sql` is not a mirror of the Raku solution because PostgreSQL provides a `generate_series()` function that can be used to generate a range of dates, in this case from the very beginning to the very end of the year incrementing one day at a time.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
  f_working_days_per_year( yy int default extract( year from current_date ) )
  RETURNS int
AS $CODE$
  SELECT count( v )
  FROM generate_series( make_date( yy, 01, 01 ),
                       make_date( yy, 12, 31 ),
                       '1 days' ) v
  WHERE
     extract( dow from v ) NOT IN ( 0, 6 );
  $CODE$
  LANGUAGE sql;
                                                             
                                                             
```
<br/>
<br/>

It does then suffice to count all the resulting rows (i.e., days) that are not Sundays or Saturdays (respectively 6 and 0 in `DOW`) to get the final result.

<a name="task2pg"></a>
## PWC 138 - Task 2 in PostgreSQL `plpgsql`

The second task is a `plpgsql` re-implementation of the Raku solution.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
  f_split_numbers( n int default 9801 )
  RETURNS int
AS $CODE$
  DECLARE
  sqrt int := sqrt( n );
  digits int[] := regexp_split_to_array( n::text, '' );
  aggregation int := 0;
  sum_left int := 0;
  sum_right int := 0;
BEGIN
  RAISE DEBUG 'Operating for % (sqrt = %)', n, sqrt;

  FOR aggregation IN 1 .. length( n::text ) LOOP
    RAISE DEBUG 'Aggregation index %', aggregation;

    SELECT array_to_string( digits[1:aggregation], '' )::int
           , sum( r )
      FROM ( SELECT unnest( digits[ aggregation + 1: length( n::text ) ] ) AS r ) rr
      INTO sum_left, sum_right;

    RAISE DEBUG '% + %', sum_left, sum_right;

    IF ( sum_left + sum_right ) = sqrt THEN
      RETURN 1;
    END IF;
  END LOOP;

  RETURN 0;
END
  $CODE$
  LANGUAGE plpgsql;


                                                                                                                                                                                          

```
<br/>
<br/>

There is a cast here and there from `int` to `text` and viceversa to make the whole thing work, but the idea is to split the given number into its digits (text) and then aggreate them from left to right. The difference from the Raku implementation is that, in SQL, *arrays start at index 1*. Please note also the use of `unnest`, that translates an array into a tabular data, so that the `sum` function can be used.
