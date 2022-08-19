---
layout: post
title:  "Perl Weekly Challenge 178: damn numbers again!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 178: damn numbers again!

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 178](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0178/){:target="_blank"}.

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
## PWC 178 - Task 1

Implement a *Quarter imaginary* converter. This is too much complicated for my brain, that has been returned in front of a monitor back from the holidays, so I blondly used `Base::Any` as a module to do the trick:



<br/>
<br/>
```raku
use Base::Any;

sub MAIN( Int:D $what ) {
    to-base(4,2i).say;
}
 ```
<br/>
<br/>


<a name="task2"></a>
## PWC 178 - Task 2

See when a meeting of a given duration will end in the next business slot.
I placed the `$min_hour` and `$max_hour` to keep track of when the business can be hold during the day, as well as `@working_days` to keep track of which days of week are business ones. Then I do iterate over the business days adding a minute at a time, and checking if I overflow the current business day, going then to the next one.
When I've added all the minutes, I end the looping and print out the new business hour.
<br/>
Last thing to note is that in the beginning I parse the input string to make it comprehensive to `DateTime`.


<br/>
<br/>
```raku
sub MAIN( Str $ts = '2022-08-01 10:30', Rat $duration = 4.0 ) {

    $ts ~~ /(\d ** 4) '-' (\d ** 2) '-' (\d ** 2) \s+ (\d ** 2) ':' (\d ** 2)/;

    my $when = DateTime.new: year => $/[0],
               month => $/[1],
               day => $/[2],
               hour => $/[3],
               minute => $/[4];

    my ($min_hour, $max_hour) = (9, 18);
    my @working_days = 1 .. 5;

    my $remaining_minutes = $duration * 60;

    while ( $remaining_minutes > 0 ) {


        if ( $min_hour > $when.hour > $max_hour
             || ( $when.hour == $max_hour && $when.minute >= 0 )
             || $when.day-of-week !~~ any( @working_days ) ) {
            # go to the next working day

            $when = $when.clone( hour => $min_hour, minute => 0 ).later( days => 1 );
            while ( $when.day-of-week !~~ any(@working_days) ) {
                $when = $when.later( days => 1 );
                }
        }
        else {
            # add one minute
            $when = $when.later( minutes => 1 );

            $remaining_minutes--;
        }

    }

    say $when;

}

```
<br/>
<br/>


<a name="task1plperl"></a>
## PWC 178 - Task 1 in PostgreSQL PL/Perl

Not implemented this week!

<br/>
<br/>

``` sql
```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 178 - Task 2 in PostgreSQL PL/Perl


Not implemented in this week!

<br/>
<br/>

``` sql

```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 178 - Task 1 in PostgreSQL PL/PgSQL

Not implemented this week!

<br/>
<br/>

``` sql
```
<br/>
<br/>




<a name="task2plpgsql"></a>
## PWC 178 - Task 2 in PostgreSQL PL/PgSQL

Not implemented this week!

<br/>
<br/>

``` sql

```
<br/>
<br/>
