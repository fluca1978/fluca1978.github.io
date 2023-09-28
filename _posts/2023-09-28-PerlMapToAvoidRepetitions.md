---
layout: post
title:  "Perl map to Avoid Code Repetition"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
A simple consideration to emphasize how `map` can help you shorten your code.

# Perl map to Avoid Code Repetition

I see new developers often having difficulties in understanding how operators like `map` can be useful.
While it is true that `map` can be useful to transform an array into another array, *remapping* the former into the latter, the `map` can be used also to automate the same code over and over.


Imagine you need to extract a set of values calling the same method with different parameters:

<br/>
<br/>
```perl
my $a = $xml->getTag( 'a' );
my $b = $xml->getTag( 'b' );
my $c = $xml->getTag( 'c' );
```
<br/>
<br/>

Perl support mutiple assignments, so the above can be rewritten as:

<br/>
<br/>
```perl
my ( $a, $b, $c ) = ( $xml->getTag( 'a' ),
                      $xml->getTag( 'b' ),
					  $xml->getTag( 'c' ) );
```
<br/>
<br/>

The above is the first step towards using `map` to evolve the above boring code:

<br/>
<br/>
```perl
my ( $a, $b, $c ) = map { $xml->getTag( $_ ) }
                    qw/ a b c /;

```
<br/>
<br/>

Note how in the above piece of code, there is a single call to the method, while the work is repeated by the `map` operator that will invoke the method for every word in the `qw` (quote word) operator. The calls (return value) are then placed into a list, that is in turn assigned to variables.
