---
layout: post
title:  "Perl5 -> Perl 6: Schwartzian transform"
author: Luca Ferrari
tags:
- perl6
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
How can we use this powerful idiom in the Perl 6 language?

## Perl5 -> Perl 6: Schwartzian transform
-----

The [Schwartzian transform](https://en.wikipedia.org/wiki/Schwartzian_transform) is a real powerful idiom that, exploiting a concatenation of ```map``` and ```sort``` allows for a quick and short reordering of a complex data structure.

Before keep reading, please consider the Perl 6 operators are more efficient, lazy and smart than their Perl 5 counterparts. This means that
[you could not need the Schwartzian transform at all!](https://www.learningperl6.com/2016/12/13/quick-tip-28-perl-6s-schwartzian-transform/)



Now imagine you have an hash containing the person name as key and the score of the game as value. How can you sort the hash
(keys) from the lowest to higher score? It is quite easy using the Schwartzian transform:

```perl
my %scores_of = ( luca => 10,
                  simon => 5,
                  fred => 90,
                  barnie => 45,
    );


my @sorted_keys = map { $_->[ 1 ] }
                  sort { $b->[0] <=> $a->[0] }
                  map { [ $scores_of{ $_ }, $_ ] }
                  keys %scores_of;

say "Sorted keys @sorted_keys";
```

Allow me to do a quick walkthru (please consider that you should read it from right to left):

1. extract the keys of the hash thru the ```keys``` operator;
2. pass each key as topic variable to ```map``` and build a single tuple as an array reference (thru ```[]```), having
the value (i.e., the score) as elemnt *0* and the name as element *1*;
3. pass each tuple returned as array of arrayref from ```map``` to ```sort```. Since we want to order from the greatest to the lower,
i.e., inverse natural ordering, we compare ```$b``` first against ```$a```. The ```sort``` operator will return a list of tuples (arrayref)
ordered;
4. remap the tuples keeping only the element at index *1*, that is, the hash key (i.e., the name).

How does it translate to Perl 6? Quite easy taking care of a couple of suggestions:
- it have to be read from left to right, we are working on objects now!
- there is no need to use ```[]``` since comma now produces a list (and ```sort``` know uses objects);
- [```sort```](https://docs.perl6.org/routine/sort) does not use anymore global variables ```$a``` and ```$b```, so you have to declare
them explicitly in the block signature.

Therefore, here it is the code:

```perl
my @sorted_keys = %scores_of{}:k
    .map( {  %scores_of{ $_ }, $_   } )
    .sort( -> $a, $b { $b[ 0 ] <=> $a[ 0 ] } )
    .map: { $_[ 1 ] };

@sorted_keys.say;
```

Here's the walkthru:
1. extract the keys thru the ```:k``` adverb on the whole hash slice;
2. ```map``` the keys as a list of tuples as in the above (note the square brackets are not mandatory here);
3. ```sort``` the keys picking them a couple at a time and assigning to ```$a, $b```;
4. ```map``` the result keeping only the field at index *1* (note that here there's no array ref but just a single array).

And this is the general way of translating the Schwartzian transform bit by bit, but as already written [and presented here](https://www.learningperl6.com/2016/12/13/quick-tip-28-perl-6s-schwartzian-transform/) Perl 6 operators are a lot more **smart** then their Perl 5 counterpart, so chances are you will never need the Schwartzian transform at all!
