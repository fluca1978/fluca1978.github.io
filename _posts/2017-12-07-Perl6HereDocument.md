---
layout: post
title:  "Perl5 -> Perl 6: here document"
author: Luca Ferrari
tags:
- perl6
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
Perl 6 enhances the *here document* support.

# Perl5 -> Perl 6: here document

In Perl 5 the *[here document](https://docs.perl6.org/language/quoting#Heredocs:_:to)* syntax is quite simple and straightforward, as in a shell-like interface:

```perl
say << "END_INSTRUCTIONS";
Dear user,
in order to do so and so you
have to buz and foo.
END_INSTRUCTIONS
```

The rules are simple: if the delimiter is within double quotes it can interpolate variables, following the interpolation rules. The delimiter
has to be at the very beginning of the line, or the parser will get confused.

How does the Perl 6 enhance this?

First of all the `:to` adverb can be used on any quoting construct to inform the parser that you are applying the *here document*.
This also allows for using the standard method interface allowing to end the current line of code and concatenating it to another method call.

Let's see a simple example:

```perl
my $nuke = 'rm -rf /';
say( q:to/END_OF_NUKE/ );
If you want to nuke your own installation
try to run the following command:
     $nuke
and good luck!
END_OF_NUKE
```

This produces the following result:

```shell
If you want to nuke your own installation
try to run the following command:
$nuke
and good luck!
```

As you can see, this is not going to interpolate the `$nuke` variable since the `q` construct does not interpolate.
A better result can be obtained using `qq` which interpolates.
Moreover, in Perl 6 you can move the marker from the beginning of the line: the number of spaces will be removed from every row of the here document content.
Last, you can multi-document content at once, and therefore the following example:

```perl
say( q:to/END_OF_BACKUP/, qq:to/END_OF_NUKE/ );
   Please consider do a full backup
   of your system before proceeding.
 END_OF_BACKUP
If you want to nuke your own installation
try to run the following command:
$nuke
and good luck!
END_OF_NUKE
```

produces the following content:

```shell
  Please consider do a full backup
  of your system before proceeding.
If you want to nuke your own installation
try to run the following command:
rm -rf /
and good luck!
```

Note that both interpolation and reformatting removing the leading white spaces (partially, that is intentional) has been done.
But this syntax allows also for some more elaborate, like assignment:

```perl
my ( $backup_text, $nuke_text ) = q:to/END_OF_BACKUP/, qq:to/END_OF_NUKE/;
     Please consider do a full backup
     of your system before proceeding.
     END_OF_BACKUP
     If you want to nuke your own installation
     try to run the following command:
     $nuke
     and good luck!
     END_OF_NUKE
```

or method chaining:

```perl
say( q:to/END_OF_BACKUP/.uc, qq:to/END_OF_NUKE/.uc );
...
```
