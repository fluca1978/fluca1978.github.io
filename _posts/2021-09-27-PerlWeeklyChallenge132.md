---
layout: post
title:  "Perl Weekly Challenge 132: not so clear..."
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 132: not so clear...

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 110](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0110/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)



<a name="task1"></a>
## PWC 132 - Task 1

Both the tasks for this weekly challenge are not so clear to me, so I could have completely misunderstood the aims. However, the first task was about finding out the *mirror dates* of a given one, that is given an input date (assumed as your birthday), compute two other dates:
- a date in the past, that is the birth of a person that on the given input date has the same age of you;
- a date in the future that is when a person born on today will have the same date as you.

<br/>
I decided to implement this by means of using `Date` and computing days:The code therefore looks like:

<br/>
<br/>
```raku
multi sub MAIN( Str $date where { $date ~~ / \d ** 4 '/' \d ** 2 '/' \d ** 2 / } ) {
    $date ~~ / (\d ** 4 ) '/' (\d ** 2 ) '/' ( \d **2 ) /;
    MAIN( $0 ~ '-' ~ $1 ~ '-' ~ $2  );
}

multi sub MAIN( Str $date where { $date ~~ / \d ** 4 '-' \d ** 2 '-' \d ** 2 / } ) {
    my $birthday = Date.new: $date;
    my $today = Date.today;
    my $days = $today - $birthday;
    my @dates = $birthday - $days, $today + $days;
    @dates.join( ', ' ).say;
}

```
<br/>
<br/>

The program accepts dates written with either a `/` or a `-`. Depending on the input date, it is computed the person `$birthday`, and then it is computed the number of days that differ from `$today`. Hence, it is trivial to get the birthday in the past and in the future by simply adding `$days` or subtracting them to the appropriate starting date.
<br/>
The results from this script are slightly different than those proposed in the solution of the task, and I assume this is due to different form of computations. For example, one could be more accurate computing the number of seconds instead of the number of days.



<a name="task2"></a>
## PWC 132 - Task 2

The second task was about producing an ugly hash join. I state "ugly" because it does not have to remove duplicates, and it must be done in stages: there is an upper limit that is how many record to keep in memory. Once the memory is full, there must be the scan of the outer set of data. This does not sound to me as an hash join, since there is no hashing, but again, I could be wrong on the interpretation of the assigned task.
<br/>
The code looks like:

<br/>
<br/>
```raku
class HashTable {
    # hash of the table
    has Int $.size = 2;

    method !doMatch( @memory, @S, $r-index, $s-index ) {
        for @memory -> $rr {
            for @S -> $s {
                say $rr.join( ' ' ) ~ $s[ 0 .. $s-index - 1, $s-index + 1 .. * - 1 ],join( ' ' )  if $s[ $s-index ] ~~ $rr[ $r-index ];
            }
        }

        @memory = ();
    }

    method match( @R, @S, Int $r-index = 0, Int $s-index = 0 ) {
        my @memory;
        for @R -> $r {
            @memory.push: $r;
            self!doMatch( @memory, @S, $r-index, $s-index )  if @memory.elems == $!size;
        }

        self!doMatch( @memory, @S, $r-index, $s-index ) if @memory;
    }
}

sub MAIN() {
    my  @player_ages = 
    [20, "Alex"  ],
    [28, "Joe"   ],
    [38, "Mike"  ],
    [18, "Alex"  ],
    [25, "David" ],
    [18, "Simon" ],
    ;

    my  @player_names = 
        ["Alex", "Stewart"],
        ["Joe",  "Root"   ],
        ["Mike", "Gatting"],
        ["Joe",  "Blog"   ],
        ["Alex", "Jones"  ],
        ["Simon","Duane"  ],
    ;

    my $hash = HashTable.new;
    $hash.match( @player_ages, @player_names, 1, 0 );
}

```
<br/>
<br/>

The class `HashTable` is the core of the program: it has a `$!size` that is how many elements must be kept into memory at once. The `match` method performs a loop over `@R` to call in batches the `doMatch` method, that in turn does a full scan of `@S`. It would be possible to store `@memory` has an `HashTable` property, so to avoid some arguments in the method calls, but this does not change the behavior.
<br/>
The `doMatch` method is then called a last time, in the case the size of `@R` is not a multiple of `$!size`.
