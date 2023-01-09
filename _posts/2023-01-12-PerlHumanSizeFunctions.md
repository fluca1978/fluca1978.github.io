---
layout: post
title:  "Fancy ways to get human file sizes in Perl"
author: Luca Ferrari
tags:
- Perl
permalink: /:year/:month/:day/:title.html
---
How to manipulate arrays in a fancy way to get human file size(s) in Perl.

# Fancy ways to get human file sizes in Perl

Often, in my applications, I need to get a *human readable* file size. This is quite simple to implement in Perl by means of a function, for instance something like the following:

<br/>
<br/>
```perl
use constant KB => 1024;
use constant MB => 1024 * 1024;
use constant GB => 1024 * 1024 * 1024;
use constant TB => 1024 * 1024 * 1024 * 1024;


sub human_size {
  no warnings;
  my ( $size ) = $_[0];
  given( $size ) {
    when ( $_ >= TB ) { sprintf "%.2f TB", $_ / TB; }
    when ( $_ >= GB ) { sprintf "%.2f GB", $_ / GB; }
    when ( $_ >= MB ) { sprintf "%.2f MB", $_ / MB; }
    when ( $_ >= KB ) { sprintf "%.2f kB", $_ / KB; }
    default           { sprintf "%d bytes", $_;     }
  }
}

```
<br/>
<br/>

The idea is simple enough: the function accepts the bytes as an argument, therefore something you can get by means of the `-s` Perl builtin operator. The function uses the `given when` construct to compute the size in a human readable way, and presents the user with a string.


## A *fancy* approach

There is another *fancy* way to compute the human readable size, this time starting from only the *names* of sizes.

<br/>
<br/>
```perl
use constant SIZE_NAMES => qw/ tera giga mega kilo /;

sub human_size_fancy {
    my ( $size ) = @_;
    my @scale = (SIZE_NAMES);
    while ( $size < ( 1024 ** @scale ) ) {
	   shift @scale;
    }

    sprintf '%.2f %sbytes', ( $size / ( 1024 ** @scale ) ), $scale[ 0 ];
}

```
<br/>
<br/>

The idea is as follows:
- the `SIZE_NAMES` constant acts like a list of prefixes, note that the base unit `bytes` is not included in the list;
- the `@scale` array is a clone of the constant, that is used in order to be manipulated (if needed) every time the functio is invoked;
- the `while` loop removes the head of the `@scale` array until the `$size` variable (in bytes) is greater than the number of the power of `1024` by the number of elements in the `@scale`. This means that in the beginning, with all the elements, the loop is looking for `$size` to be greater than `1024 ** 4`, i.e., one gigabyte, and so on;
- when the loop finishes, the `@scale` array contains the list of size prefixes greater than the actual `$size`, therefore the `$scale[0]` is the greatest prefix that can be used;
- the `sprintf` does the printing reducing the `$size` by means of the `1024` power by the number of elements of the `@scale`. In the case the array is empty, only `bytes` is printed and no division is performed (or better, a division by `1` is done).


## A more *fancy* approach

There is also the capability to use another, more compact, approach, that do not require to manipulate an intermediate array.

<br/>
<br/>
```perl
sub human_size_more_fancy {
    my ( $size ) = @_;
    for my $i ( 0 .. scalar (SIZE_NAMES) ) {
		next if $size < ( 1024 ** ( scalar (SIZE_NAMES) - $i ) );
		return sprintf '%.2f %sbytes', ( $size / ( 1024 ** ( scalar (SIZE_NAMES) - $i ) ) ), (SIZE_NAMES)[ $i ];
    }
}

```
<br/>
<br/>

The idea here is to traverse the list of `SIZE_NAMES` starting from the greatest, and go down one order of magnitude at a time while the `$size` is greater than the order of magnitue itself. As soon as the `$size` becomes smaller than the current order of magnitude, the function returns the human readable string. Note that the final size is computed with `1024` power by the length of `SIUZE_NAMES` *minus* the dropped order of magnitude `$i`.

# Conclusions

Perl, as usual, is amazing, and the only limit is your fantasy.
But before you start writing *fancy* code, please consider the mental sanity of people coming after you, that could easily confuse what you believe is `fancy` with what they consider as `trash`!
<br/>
The above examples are available [on my gitlab repository](https://gitlab.com/fluca1978/fluca1978-coding-bits/-/blob/master/perl/human_file_size.pl){:target="_blank"}.
