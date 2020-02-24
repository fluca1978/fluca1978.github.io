---
layout: post
title:  "Perl Weekly Challenge 49: LRU and Smallest Multiples made by 1 and 0"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 49: Survivors and Palindrome Dates

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 49](https://perlweeklychallenge.org/blog/perl-weekly-challenge-049/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<a name="task1"></a>
## PWC 49 - Task 1

The first task required to find out the smallest multiple of a given number, with the restriction that the multiple number must be made only by `1`s and `0`s.
My personal solution, even if not very elegant, involved a simple `for` loop and a regular expression matching:

```perl6
sub MAIN( Int :$number! where { $number > 0 } ) {
    "Minor multiple of $number made up by 0 and 1 only is { $number * $_ }".say && exit if ( $number * $_ ) ~~ / ^ <[01]>+ $ /
    for 2 .. Inf;
}
```

It reads as follows:
- `for` every number from `2` (thus excluding the number itself) to infinity;
- the multiple is `$number * $_`;
- if that result matches against a regular expression that imposes only `1` and `0` from begin to end I print the result and issue an `exit` that terminates the program (and the loop).





<a name="task2"></a>
## PWC 49 - Task 2

This was a little more long to implement: an Least Recently Used cache.
<br/>
I decided to implement it via a specific class, named (you guess!) `LRU`:

```perl6
class LRU {
    # contains a list of Pairs (not an hash!)
    # where every pair has the value of times it has been used
    # and every key is the number
    has Pair @!values;
    has Int $.capacity;


    method get( Int $what ) {
        # find the first $what value and see where it is within the
        # array
        my ( Int $where, Pair $pair ) = @!values.first: { .key == $what }, :kv;

        # not found
        return -1 if ! $pair;

        # now remove the element from the array and
        # insert it on the right incremented
        @!values.splice( $where, 1 );
        @!values.append: $pair.key => $pair.value + 1;

        # here $pair still holds the old value
        return $pair.value;
    }

    # Is the cache full?
    method !is-full() { return @!values.elems >= $!capacity; }

    # remove the least recently used value, always the one on the left
    method !make-space() { @!values.splice( 0, 1 ); }

    method set( Int $value, Int $how-many-times ) {
        # make some space if the cache is full
        self!make-space() if self!is-full;
        @!values.push: $value => $how-many-times;
    }


    method Str() {
        my $string = "\n[Capacity: {$!capacity}]\n[Least Recently Used] -->>>";
        $string = "%s\t%d (used %d times)".sprintf( $string, $_.key, $_.value)  for @!values;
        $string ~= " <<<-- [Most Recently Used]\n";
        return $string;
    }
}

```

The class has an array named `@!values` that contains a list of `Pair`s, each one with the number to store in the cache as the key and the number of times it has been accessed as the value.
<br/>
When we require a new number, method `get()`, the array searches for such an existant `Pair` in it (or better, it searches for the `first` one `Pair`). If nothing has been found, the special value `-1` is returned, otherwise there is some work to do. The `Pair` is removed from the array, and a new `Pair` with the same key is inserted at the end of the array (the end meaning the most recently used value). Then, the old pair value is returned.
<br/>
The `set()` method is somehow simpler: it pushes a new `Pair` object to the end of the array. However, in the case the array is full, the `make-space()` method removes the very first element of the array (meaning the least recently used one).
<br/>
The rest of the code is boilerplate.
<br/>
Therefore, my LRU cache uses an array where the first element is the least recently used, and the last element is the most recently used.

### Update 2020-02-24

Of course, there is no need to use `splice()` and friends, but since I was running out of caffeine, I made my life too much complex.
As shown [in this commit](https://github.com/fluca1978/perlweeklychallenge-club/commit/c60c42d569995cc87f37a2171fe09f153e8ddaf1){:target="_blank"}, it is possible to simply delete the element from the array and append it to a fresh *all-defined* entries list. Similarly, the methods `is-full()` and `make-sapce()` can be removed to avoid code verbosity; I placed them there in a first implementation where I was prototyping the `LRU` object as an hash container.
