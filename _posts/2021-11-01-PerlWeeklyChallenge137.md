---
layout: post
title:  "Perl Weekly Challenge 137: palindrome sums on long years"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 137: palindrome sums on long years

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 137](https://perlweeklychallenge.org/blog/perl-weekly-challenge-137/){:target="_blank"}.

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
## PWC 137 - Task 1

The first task was about finding out if a given year is a *long year*, meaning an year with *53* weeks instead of the "ordinary" 52. This is a piece of cake using the `Date` builtin class and its `week-number` method:

<br/>
<br/>
```raku
sub MAIN() {
    for 1900 .. 2100 -> $year {
        $year.say if Date.new( '%04d-12-31'.sprintf( $year ) ).week-number == 53;
    }
}
```
<br/>
<br/>

It could have also been written in a single line, using the postfix `for`, but the idea remains the same: iterate on every year, build a  `Date` object for the last day of the year, and get the week number via the `week-number` method.




<a name="task2"></a>
## PWC 137 - Task 2

In this task, given an integer value greater than `19` (that means, with at least two digits), we need to see if it is a *Lychrel* number. The idea is that, given a number, you keep adding it with its palindrome and see if the result is palindrome too. If it is, stop, the number is a Lychrel one, otherwise sum the result to its palindrome and go further.

<br/>
<br/>
```raku
sub MAIN( Int $n where { 10 <= $n <= 10000 }, Bool :$verbose = False )  {

    my ( $result, $iteration ) = $n,0;
    while ( $result < 10_000_000 && $iteration < 500 ) {
        $iteration++;
        $result += $result.split( '' ).reverse.join;
        if $result == $result.split( '' ).reverse.join {
            '0'.say;
            "Found $result after $iteration iterations".say if $verbose;
            exit;
        }
    }

    '1'.say;
    "Cannot find Lychrel number for $n".say if $verbose;
}
```
<br/>
<br/>

The `$result` contains the result of summing a number with its palindrome at every step. To get quickly the palindrome of a number I do use `split` to get its digits, `reverse` to reverse the array, and `join` to re-assemble it. It is not the only way, it can be used also `flip` on a `Str`, or other techniques.
<br/>
The `$verbose` flag allows for printing of some messages about the sattus of the computations.


<a name="task1pg"></a>
## PWC 137 - Task 1 in PostgreSQL `plpgsql`

The implementation in `plpgsql` is a mirror of the implementation in Raku:

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
f_long_year()
RETURNS SETOF int
AS $CODE$
DECLARE
        current_year int;
BEGIN
        FOR current_year IN 1900 .. 2100 LOOP
           IF date_part( 'week', make_Date( current_year, 12, 31 ) ) = 53 THEN
              RETURN NEXT current_year;
           END IF;
        END LOOP;

        RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The trick here is to use `date_part`, that is a *magical* function that allows for extracting different information from a date. The date is built with `make_date`, that accepts the year, the month and the day. It is not mandatory, since a string concation is interpreted in the right sense in PostgreSQL, but using  `make_date` makes the function easier to red.
<br/>
It is also important to note that the `for` loop does a `RETURN NEXT`, that is kind of *yeld*: it allows the function to append a new value to the result set and proceed further with the looping. This allows the client to get results as a *stream* and the function to not store all of them while performing the computation.

<a name="task2pg"></a>
## PWC 137 - Task 2 in PostgreSQL `plpgsql`

The second task is a `plpgsql` re-implementation of the Raku solution.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
f_lychrel( n int, verb boolean default false )
RETURNS smallint
AS $CODE$
DECLARE
        result    bigint := n;
        iteration int    := 0;
BEGIN
        IF n < 10 OR n > 10000 THEN
           RAISE 'n is out of bounds!';
        END IF;

        WHILE result < 10000000 AND iteration < 500 LOOP
              iteration = iteration + 1;
              result    = result + reverse( result::text )::int;
              IF result = reverse( result::text )::int THEN
                 IF verb THEN
                    RAISE INFO 'Found % after % iterations', result, iteration;
                 END IF;

                 RETURN 0;
             END IF;

        END LOOP;

        RETURN 1;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

In the case of `plpgsql` we can exploit the `reverse` function that does the flipping of a given number, converted as a `text` (i.e., a string) and then reconvert the result as a number.
<br/>
It is interesting to note how the `plpgsql` becomes quickly longer than the Raku implementation, and this is due to the syntax of conditionals but also to the fact that we need to check arguments within the core of the function, instead of being able to declare complex conditions on the argument list.
