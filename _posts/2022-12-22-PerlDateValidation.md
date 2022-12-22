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

In this article I present a couple of ways to quickly and simply validate dates on embedded programs.

## Using `DateTime` and `Try::Tiny`

One approach that I use very often, is to combine a regular expression with `DateTime` to build a date object and let the module itself do the validation. Unluckily, `DateTime`*die*s when the date is wrong, so I need to catch the error. I personally use `Try::Tiny` to catch errors, so that the code for the date validation appears as the following:

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
Using `Try::Tiny` is not mandatory, since it is possible to `eval` the building of the object and inspect `$@`, but I think `Try::Tiny` provides a more readable code.


## Using `DateTime::Format::Strptime`

The `DateTime::Format::Strptime` is a module that accepts a format string and tries to parse the value provided. If the parsing is fine, a valid `DateTime` object is returned, otherwise an undefined value is provided.

<br/>
<br/>
```perl
if ( $current_date ) {
  my $tf = DateTime::Format::Strptime->new(
  					   pattern => '%d%m%Y',
  					   strict => 1,
  					   on_error => sub {
  					       my ( $object, $error ) = @_;
  					       say "Cannot parse $current_date : $error" if $verbose;
  					       },
  					  );
  my $when = $tf->parse_datetime( $current_date );
  $current_date = undef if ! $when;
  $current_date = sprintf( "%02d-%s-%04d", $when->day, uc( $when->month_abbr ), $when->year ) if $when;
}

```
<br/>
<br/>

The above piece of code does pretty much the same stuff in the previous example.
The idea is that the `DateTime::Format::Strptime` object is created with an `on_error` value set to a subroutine reference. When the module catches an error in parsing a date, it will call the `on_error` subroutine, that in turn can decide to `die` or complete in some way.
The parsing of the date happens when the `parse_datetime` method is invoked, and the input of such method is the string that you want to parse. The result is the `DateTime` object, that you can use as usual.


# Conclusions

Parsing dates is, apparently, very simple thanks to the powerful regular expression engine embedded into Perl.
However, there are edge cases where a carefully crafted regular expression could fail to discover semantic errors, therefore it is always better to provide a formal validation by means of a *dateish* object.
