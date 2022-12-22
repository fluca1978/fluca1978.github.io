---
layout: post
title:  "Perl: Validating Dates"
author: Luca Ferrari
tags:
- perl

permalink: /:year/:month/:day/:title.html
---
A simple approach to validate dates.

# Perl: Validating Dates

When writing Perl applications, I often have the problem of validating dates.
<br/>
Regular expressions are a great tool to validate a date format, but I often find out that the semantic values of the date are wrong, for example *30th of February*.
<br/>
One approach that today I take very often, is to combine a regular expression with `DateTime` to build a date object and let the module itself do the validation. Unluckily, `DateTime`*die*s when the date is wrong, so I need to catch the error. I personally use `Try::Tiny` to catch errors, so that the code for the date validation appears as the following:

<br/>
<br/>
```perl
if ( $current_date && $current_date =~ /^ (?<day>\d{2}) (?<month>\d{2}) (?<year>\d{4}) $/x ) {
 try {
     my $when = DateTime->new( year => $+{ year }, month => $+{ month }, day => $+{ day } );
     $current_date = sprintf "%02d-%s-%04d", $when->day, uc( $when->month_abbr ), $when->year;

 }
 catch {
     say "Cannot parse $current_date" if $verbose;
     $current_date = undef;
 }
}
else {
 $current_date = undef;
}
```
<br/>
<br/>

First of all I check the `$current_date` against a regular expression capturing the usual stuff, the year, the day and the month. Then I do pass the information to `DateTime` to build a correct object, and in the case I fail, the `catch` block of code is executed. Otherwise I do format the date in the way I need.
In all other cases, the `$current_date` is set to an undefined value to indicate the date is extremely wrong.
<br/>
<br/>
It is surely possible to make the above piece of code nicer and shorter, but it is shown as it is just to show to you the main concept of how to correctly parse dates.
