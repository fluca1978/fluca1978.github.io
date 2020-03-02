---
layout: post
title:  "Perl Weekly Challenge 50: overlapping ranges and nobel numbers"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 50: overlapping ranges and nobel numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 50](https://perlweeklychallenge.org/blog/perl-weekly-challenge-050/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<a name="task1"></a>
## PWC 50 - Task 1

The first task involved overlapping ranges, unluckily the smartmatching operators checks for *containing* ranges, so while the ranges `2..7` and `3..9` are overlapping, to regard to the task, they are not in sense of the `~~`.
<br/>
My first solution was quite simple: implement a couple of functions to check if the ranges overlap and build a merging match:

```perl6
sub overlap( Range $a, Range $b ) {
    return $b.max > $a.max > $b.min || $b.max >  $a.min > $b.min;
}

sub merge( Range $a, Range $b ) {
    return Range.new: min( $a.min, $b.min ), max( $b.max, $a.max );
}
```

The only *smart* thing used here is the form `$b.max > $a.max > $b.min` that Raku allows and that other languages have to express like `$b.max > $a.max && $a.max > $b.min`.
The final merge is made by the min and max values found between the ranges.
<br/>
The other tricky part is that the list of ranges must be searched in pairs, but in the case of a merge two pairs are removed, otherwise only one is:

```perl6
while ( @ranges.elems  ) {
    if overlap( @ranges[ 0 ], @ranges[ 1 ] ) {
        # merge the ranges
        @merged-ranges.push: merge( @ranges[ 0 ], @ranges[ 1 ] );
        @ranges.shift;
        @ranges.shift;
    }
    else {
        # dont' merge, push the first one
        @merged-ranges.push: @ranges[ 0 ];
        @ranges.shift;
    }
}

```

Please note that, in the case there is an overlap, both ranges are *shifted* out of the array, otherwise only the first one is removed. This makes the next iteration continuing from exactly the second range that did not match.
<br/>
<br/>
However, it is possible to write the whole thing in a way that is *less readable* but much more compact:

```perl6
while ( @ranges.elems  ) {
    my $a = @ranges.shift;
    my $b = @ranges[ 0 ];
    @merged-ranges.push( Range.new( min( $a.min, $b.min ), max( $a.max, $b.max ) ) )
               && @ranges.shift
               && next
              if $b.max > $a.max > $b.min || $b.max >  $a.min > $b.min;

    @merged-ranges.push: $a;
}

```

The first range is always shifted out of the array, since it will either be merged or pushed as it is into the final result array. Then the trick is about the postponed `if`: in the case there is a merge a new `Range` is build, and also the second range is shifted out of the array and the iteration proceeds to the next cycle.
Otherwise, the first range is pushed as it is. The usage of `&& next` represents a dirty way of implementing a postfix `else`.

<a name="task2"></a>
## PWC 50 - Task 2

The second task was about Nobel numbers: given a list a number is a Nobel one if there exactly the same number of its value greater than it in the list.
<br/>
The first solution was quite straightforward:

```perl6
my @L = (1..50).List.pick: $how-many;

# first approach: use a grep to count how
# many elemnts are there
my @nobel;
for @L -> $a {
    @nobel.push( $a ) if @L.grep( { $a < $_  } ).elems == $a;
}
```

First of all, I convert a range into a `List` to use the `pick` method that, in a list accepts the number of elements to randomly picks.
<br/>
After that, I perform a loop over all the elements and `grep` all the elements greater than the one I'm inspecting. If the `grep`ped list has a number of elements that match the value I'm inspecting, such element is appended to the list of `@nobel` numbers as final result.
<br/>
However, this is not the only one approach: if you order the list before you *scan* for values, you can perform a little smarter approach. In fact, once the list is sorted, the number at position `x` must have exactly other  elements after it that matches its value, so:

```perl6
@L .= sort;
for 0 .. @L.elems - 1 {
    @nobel.push( @L[ $_ ] ) if @L[ $_ ] == @L.elems - $_;
}
```
