---
layout: post
title:  "Perl5 -> Perl6: a file into an array"
author: Luca Ferrari
tags:
- perl6
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
A simple snippet about some idiom I carry on from my Perl 5 day-to-day work

## Perl5 -> Perl6: a file into an array
-----

One of the most common task I find in my Perl programs is to read data from text files, and more often that what I would expect, I need to
slurp the file (read all the lines at once). Well, to be more precise, I need to read all content into an array (while *slurping* is to read
all the content into a scalar).

In Perl 5 I tend to use the following snippet:

``` perl
 my @headers;
 if ( @ARGV > 1 ){
     open my $tag, "<", $ARGV[ 1 ]   || die "\nError\n";
     local $/ = "\r\n";
     chomp( @headers = <$tag> );
     unshift @headers, '#NUM' if ( @headers );
     close $tag;
 }

```

The idea is as simple as:
1. define the ```@headers``` empty list;
2. check if the name of the file to read has been passed as argument;
3. open such file;
4. change the input field separator (DOS mode), not always required;
5. read all lines into ```@header``` and clean each line;
6. place other piece of data into the array;
7. close the file handle.

In Perl 6 the above reduces as:

``` perl
 my @headers;
 if ( $file_tag.defined ) {
     @headers = $file_tag.IO.lines( :chomp );
     @headers.unshift( '#NUM' ) if ( @headers );
 }

```
First of all, using the named parameters to the ```MAIN``` function I do not have to check for the arity of arguments, but can
check on the argument itself (in this case ```$file_tag```).
The ```lines``` method allows to get all the content at once, or well, *lazyly*, but since assigning a ```Seq``` to an array
*reifies* it, it works as reading all the file and putting the lines into the array without any risk of leaving a pending file handle or missing
any line. It is interesting to note that the ```lines``` method allows a boolean attribute, ```chomp``` to perform the clean up of each line.
Since the method delegates to ```IO::Handle.lines```, and since by default the latter uses both ```"\n"``` and ```"\r\n"``` to clean up lines, all the magic is done for me.
Most notably, the ```chomp``` attribute is true by default, so I would have not declared it at all!
The remaining is quite obvious.`
