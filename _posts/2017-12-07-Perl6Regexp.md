---
layout: post
title:  "Perl5 -> Perl 6: glance at regexp"
author: Luca Ferrari
tags:
- raku
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
Perl 6 has a mature and feature rich regular expression engine, with several little differences from its precedessor language.

# Perl5 -> Perl 6: glance at regexp

In Perl 6 there are, obviously, regular expressions (regexp) that work similarly but not identically to the Perl 5 ones.
It is not possible to write down into a single article all the capabilities of the regexp in Perl 6, so I'm giving some hints about the features I use the most.

## Extended mode is on by default

In Perl 5 I was used to the `x` switch to enable *extended mode*, a way that allow a regexp to be made of white spaces not used against the searching for target. In other words, it was possible to insert comments, newlines (vertical spaces) and blanks (horizontal spaces) within a single regexp.

In Perl 6 this expanded mode is on by default, so that the following regular expressions are all the same:

```perl
/rm/

/ rm /

/ rm # search for a remove command
/
```

## The match operator is now `~~`

In Perl 5 a `=~` was used to anchro a regexp, in Perl 6 it is now `~~` (remember that `~` is the string concatenation). Similarly the `!~~` is the negation of a regular expression (i.e., what in Perl 5 was `~!`).


## The result of match is either Nil or Match

The interesting part is immediatly shown: if a regexp hit a match it produces a [Match](https://docs.perl6.org/type/Match) (or a Nil one).
The `Match` class provides a few interesting methods, so for instance you can easily check the original string (`orig`), what was found before and after (`prematch` and `postmatch`) and the indexes (`from` and `to`).

**The match is stored within the `$/` reference**:

```perl
my $nuke = 'rm -rf /home/luca/tmp/test.pl';

if ( $nuke ~~ / rm # search for a remove command
     / ) {
    say "Dangerous command found in " ~ $/.orig;
    say "The command is at index " ~ $/.from ~ " for other " ~ $/.to ~ " chars";
    say "Before the command there was " ~ $/.prematch;
    say "Object of the command was " ~ $/.postmatch;
}
```

The above example produces the following output:

```shell
Dangerous command found in rm -rf /home/luca/tmp/test.pl
The command is at index 0 for other 2 chars
Before the command there was
Object of the command was  -rf /home/luca/tmp/test.pl
```

## Grouping and capturing

The parentheses are used as in Perl 5 to group and capture slices of a match.
In such case, the `$/` is converted into a list of objects `Match`. It is still possible to access each single match via `$0`, `$1` and so (please note that indexes now start from zero, to be coherent with the list concept).

So for instance:

```perl
my $nuke = 'rm -rf /home/luca/tmp/test.pl';

if ( $nuke ~~ / rm \s+ ('-'\w+) \s+ (.+) / ) {
    for $/.list -> $current_match {
        say 'Current match is ' ~ $current_match;
        say '  From ' ~ $current_match.from ~ ' to ' ~ $current_match.to;
        say '  Before there was ' ~ $current_match.prematch;
        say '  After there will be ' ~ $current_match.postmatch;
    }
}
```

produces the output that follows:

```shell
Current match is -rf
  From 3 to 6
  Before there was rm
  After there will be  /home/luca/tmp/test.pl
Current match is /home/luca/tmp/test.pl
  From 7 to 29
  Before there was rm -rf
  After there will be
```

To avoid capturing, square brackets can be used. This keeps the regexp readable, at least more readable than the `(?:)` Perl 5 non capturing operator:

```perl
my $nuke = 'rm -rf /home/luca/tmp/test.pl';

if ( $nuke ~~ / rm \s+ ['-'\w+] \s+ (.+) / ) {
    for $/.list -> $current_match {
       ...
    }
}
```

produces a single capture output:

```shell
Current match is /home/luca/tmp/test.pl
  From 7 to 29
  Before there was rm -rf
  After there will be
```


## Named captures

It is possible to name a capture, as in the Perl 5 way, via an hash interface on `$/` (in Perl 5 was the `$+`). As a shortcut, the special `$<>` access the `$/` hash, so the following code can be written in two ways:

```perl
my $nuke = 'rm -rf /home/luca/tmp/test.pl';

if ( $nuke ~~ / rm \s+ $<flags>=('-'\w+) \s+ $<target>=(.+) / ) {
    for $/.hash.kv -> $match_name, $match_value {
        say "Match $match_name for $match_value ";
    }

    # direct access to the hash
    say "Match flags for $<flags>";
    say "Match target for $<target>";
}
```

that produces a double printed of the capturs:

```shell
Match flags for -rf
Match target for /home/luca/tmp/test.pl
Match flags for -rf
Match target for /home/luca/tmp/test.pl
```


## Substitution

Substitution is performed via the `s///` operator in a way that is very similar to the one of Perl 5.
Naming captures (or index based ones) can be used for the inline substitution, for instance:

```perl
my $nuke = 'rm -rf /home/luca/tmp/test.pl';

$nuke ~~ s/ $<command>=(rm)
    \s+
    $<flags>=('-'\w+)
    \s+
    $<target>=(.+) /$<command> ' not allowed!' /;

$nuke.say;
```

will replace the command with the following string:

```shell
rm ' not allowed!'
```

Please note that the substitution does take care of extra spaces, so it has not expanded mode on!
