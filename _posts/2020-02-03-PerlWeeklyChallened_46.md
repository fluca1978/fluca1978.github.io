---
layout: post
title:  "Perl Weekly Challenge 46: encoded messages and open rooms"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 46: encoded messages and open rooms

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 46](https://perlweeklychallenge.org/blog/perl-weekly-challenge-046/){:target="_blank"}.


## PWC 46 - Task 1
The solution is available [here](https://github.com/fluca1978/fluca1978-coding-bits/perl6/weekly-challenge/pwc_46_1.p6){:target="_blank"}.


Given an encoded message displayed as a matrix, try to decode it.
The message to decode was:

```
P + 2 l ! a t o
1 e 8 0 R $ 4 u
5 - r ] + a > /
P x w l b 3 k \
2 e 3 5 R 8 y u
< ! r ^ ( ) k 0
```

and the idea to decode was to get every character that, on the same column, appears more than once.
For example in the first column we have:

```
P
1
5
P
2
<
```

and the only character that appears two times is `P`, so the first message of the decoded message is `P`.

In order to decode the message, I wrote a simple routine that:
1) extract every single *line* of the matrix;
2) place every character into an array, so that the matrix become an array of arrays instead of an array of strings;
3) `grep` every submatrix (i.e., array) for a char that appears more than once. 

In particular, having `@chars` as an array of array, the following loop does the trick:

```perl6
    for @chars -> @line {
        for @line -> $searching_for {
            if @line.grep( { $_ eq $searching_for} ).elems > 1 {
                $decoded ~= $searching_for;
                last;
            }
        }
    }
```

because for every character in an array it searches for the same character with more than one occurencies, and if it founds it, the char is appended to the `$decoded` final string and the loops breaks to the next line.
<br/>
Therefore, the solution of the encoded message is **`PerlRaku`**.
<br/>


## PWC 46 - Task 2
The solution is available [here](https://github.com/fluca1978/fluca1978-coding-bits/perl6/weekly-challenge/pwc_46_2.p6){:target="_blank"}.

Given 500 hotel rooms, and 500 employees, see which rooms are still open after every employee has turned to open/close a room based on his number.
<br/>
Again, the second task was somehow simpler than the first one, at least to me.
<br/>
I created an hash of rooms, all with the status `False` to indicate the rooms were all closed:
<br/>
```perl6
my %rooms = ( 1 .. $room-count).map: * => False;
```
<br/>
Then I looped over all the employess and switched the open/close status depending on the reminder of the employee number. It was a double loop actually:
<br/>
```perl6
for 1 .. $room-count -> $employee {
    %rooms{ $_ } = ! %rooms{ $_ } if $_ %% $employee for 1 .. $room-count;
}
```
<br/>
It now becomes easy to print out which rooms are still open:
<br/>
```perl6
say "Room $_ is Open" if %rooms{ $_ } for %rooms.keys.sort: *.Int <=> *.Int;
```
<br/>
Please note that I have to explicitly force the `Int` sorting to get room numbers printed out in the right order.
<br/>
The final output is therefore:
```
% perl6 ch-2.p6
Room 1 is Open
Room 4 is Open
Room 9 is Open
Room 16 is Open
Room 25 is Open
Room 36 is Open
Room 49 is Open
Room 64 is Open
Room 81 is Open
Room 100 is Open
Room 121 is Open
Room 144 is Open
Room 169 is Open
Room 196 is Open
Room 225 is Open
Room 256 is Open
Room 289 is Open
Room 324 is Open
Room 361 is Open
Room 400 is Open
Room 441 is Open
Room 484 is Open
```
