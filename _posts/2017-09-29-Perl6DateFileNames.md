---
layout: post
title:  "Perl5 -> Perl 6: dates as strings in file names"
author: Luca Ferrari
tags:
- raku
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
One common task is to handle dates as strings when crunching files.

## Perl5 -> Perl 6: dates as strings in file names
-----

One common task I perform is to output some report information into text files (CSV, org-mode, SQL, and alike). In order to
don't mess up with several reports, I place at least the date in a *YYYY-MM-DD* format into the file name so that I can easily
list and diff them.
This is a real simple task, but it works a little differently between Perl 5 and Perl 6 due to the OOP stronger nature of the latter.

In Perl 5 it could work as follows:

```perl
my $root = "/mnt/data";
my $name = "report";


my @when = localtime;

my $file_name = sprintf "%s/%s_%04d-%02d-%02d.txt",
    $root,
    $name,
    $when[ 5 ] + 1900,
    $when[ 4 ],
    $when[ 3 ];
```

that produces the file name ```/mnt/data/report_2017-08-28.txt```. The above is the *ancient* way of doing this in Perl, while in modern
version you can do it using the ```DateTime``` object as follows:

```perl
use DateTime;
my $when = DateTime->now;
my $file_name = sprintf "%s/%s_%04d-%02d-%02d.txt",
    $root,
    $name,
    $when->year,
    $when->month,
    $when->day;

```

that of course produces the same result but in a simpler way (you don't have to remember array indexes and, most notably, to add the
century part to the years).

In Perl 6 the ```[Date](https://docs.perl6.org/type/Date)``` object is within core, so that the above code can be simply rewritten as follows:

```perl
my $when = Date.today;

my $file_name = "%s/%s_%04d-%02d-%02d.txt"
                 .sprintf( $root,
                           $name,
                           $when.year,
                           $when.month,
                           $when.day );
```

Similarly, in Perl 6 also ```[DateTime](https://docs.perl6.org/type/DateTime)``` is within core, so that you can also insert the
time of the computation as simple as follows:

```perl
my $when = DateTime.now;

my $file_name = "%s/%s_%04d-%02d-%02dT%02d-%02d.txt"
                 .sprintf( $root,
                           $name,
                           $when.year,
                           $when.month,
                           $when.day,
                           $when.hour,
                           $when.minute );
```

There is something more: both with ```Date``` and ```DateTime``` you can specify a *formatter* in order to stringify
the object without having to place placeholders within the name file. The idea is that each time the object instance has to be
stringified, the formatter will be used to return a string form of the object. Therefore:
1. we specify a *formatter* when building the object, such method will include the sprintf pattern;
2. we use the string interpolation when using the object.

```perl
my $when = DateTime.now( formatter => { "%04d-%02d-%02dT%02d-%02d"
                                            .sprintf( $_.year,
                                                      $_.month,
                                                      $_.day,
                                                      $_.hour,
                                                      $_.minute );
                                       } );


my $file_name = "%s/%s_%s.txt"
                 .sprintf( $root,
                           $name,
                           $when );

#my $file_name = "%s/%s_%s.txt"
#                 .sprintf( $root,
#                           $name,
#                           $when.Str );

```
