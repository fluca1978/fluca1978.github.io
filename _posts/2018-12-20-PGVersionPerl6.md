---
layout: post
title:  "PGVersion: a class to manage PostgreSQL Version (strings) within a Perl 6 Program"
author: Luca Ferrari
tags:
- perl6
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
While writing a program in Perl 6 I had the need to correctly parse and analyze diffefent PostgreSQL version strings. I wrote a simple and minimal class to the aim, and refactored so it can escape in the wild.

PGVersion: a class to manage PostgreSQL Version (strings) within a Perl 6 Program
---

As you probably already know, PostgreSQL has changed its versioning number scheme from a `major.major.minor` approach to a concise `major.minor` one. Both are simple enought to be evaluated with a regular expression, but I found myself wrinting the same logic over and over, so I decided to write a minimal class to do the job for me and provide several information.
<br/>
Oh, and this is Perl 6 (that I'm still learning!).
<BR/>
The class is named [`Fluca1978::Utils::PostgreSQL::PGVersion`](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/perl6/lib/Fluca1978/Utils/PostgreSQL/PGVersion.pm6) and is released as it is under the BSD Licence.

# Quick, show me something!

Ok, here it is how it works:

```perl
use Fluca1978::Utils::PostgreSQL::PGVersion;

for <10.1 11beta1 11.1 9.6.5 6.11> {
    my $v = PGVersion.new: :version-string( $_ );
    say "PostgreSQL version is $v";
    say "or for short { $v.gist }";
    say "and if you want a detailed version:\n{ $v.Str( True ) }";
    say "URL to download: { $v.http-download-url }";
    say '~~~~' x 10;
}
```

The above simple loop provides the following output:

```shell
% perl6 -Ilib usage-example-pgversion.pl
PostgreSQL version is v10.1
or for short 10.1
and if you want a detailed version:
10.1 (Major: 10, Minor: 1, stable)
Equivalent to SHOW SERVER_VERSION_NUM is 100001
URL to download: https://ftp.postgresql.org/pub/source//v10.1/postgressql-10.1.tar.bz2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PostgreSQL version is v11beta1
or for short 11beta1
and if you want a detailed version:
11beta1 (beta 1 of development branch 11)
Equivalent to SHOW SERVER_VERSION_NUM is 119999
URL to download: https://ftp.postgresql.org/pub/source//v11beta1/postgressql-11beta1.tar.bz2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PostgreSQL version is v11.1
or for short 11.1
and if you want a detailed version:
11.1 (Major: 11, Minor: 1, stable)
Equivalent to SHOW SERVER_VERSION_NUM is 110001
URL to download: https://ftp.postgresql.org/pub/source//v11.1/postgressql-11.1.tar.bz2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PostgreSQL version is v9.6.5
or for short 9.6.5
and if you want a detailed version:
9.6.5 (Major: 9.6, Minor: 5, stable)
Equivalent to SHOW SERVER_VERSION_NUM is 090605
URL to download: https://ftp.postgresql.org/pub/source//v9.6.5/postgressql-9.6.5.tar.bz2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PostgreSQL version is v6.11
or for short 6.11
and if you want a detailed version:
6.11 (Major: 6, Minor: 11, stable)
Equivalent to SHOW SERVER_VERSION_NUM is 060011
URL to download: https://ftp.postgresql.org/pub/source//v6.11/postgressql-6.11.tar.gz
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

As you can see, every version string is correctly interpreted and printed out, major and minor numbering schemes are applied depending on the version number recognized.

## From a string to a string

The class provides three *stringify* methods:
- `.gist` provides the numeric part of the version number, e.g., `9.6.5`;
- `.Str`  places a `v` (for version) in front of the numeric part of the string, e.g., `v9.6.5`;
- `.Str( Bool )` when passed a `True` value produces a verbose output explaining the version of PostgreSQL including its alfa or beta status (e.g., `9.6.5 (Major: 9.6, Minor: 5, stable)`), when invoked with a `False` argument provides the same output of `.gist`.

## Input strings

The class accepts a named argument for its construction: `version-string`. Acceptable strings are those in the form:
- *va.b.c*
- *a.b.c*
- *va.b*
- *a.b*
- *xbetay*
- *xalfay*

So for example all the following are good strings: `9.6.5`, `6.10`, `10.1`, `11beta3`.

## Main methods

The `.parse` method performs the main work disassembling the version string used to construct the object into pieces that are then stored in the class internal attributes, as numbers (`Int`). You can call `.parse` after a version object has been created to change its internal status.

<br/>
The class provides several methods with easy-to-understand names:
- `is-alfa`, `is-beta` to check if the version string identifies a development branch;
- `major-number`, `minor-number` to get the single pieces of a numbering. Please note that `.major-number` returns a `Str` because for 7 to 9 numbering scheme the major number is made by two digits separated by a dot. I'm thinking about the idea of using a `Rat` to return a numeric value, but I'm not sure it is a good idea. Internally, however, all the data is kept as integers;
- `server-version` and `server-version-num` are there to provide the same behavior of a `SHOW` issued on PostgreSQL connection;
- `http-download-url` accepts an optional URL as base and returns the HTTP link to download the specified version of PostgreSQL.

The class also defines the `ACCEPTS` method that is used for smart matching, so that you can say if two objects are the same, as well as comparing if they are older or newer with respect one to the other:

```perl
my $version-a = PGVersion.new: :version-string( '10.1' );
my $version-b = PGVersion.new: :version-string( 'v9.6.5' );
$version-a ~~ $version-b;      # False
$version-a.newer: $version-b;  # True
$version-b.older: $version-a;  # True
```

See the [tests](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/perl6/t/01-pgversion.t) for a more comphrensive usage of the instances and of the methods.

<br/>
<br/>
I hope this can be helpful to someone, I could one day collect other utilities and release them as a whole module.
