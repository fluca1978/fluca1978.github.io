---
layout: post
title:  "Perl Weekly Challenge 53: rotating matrix and vowels"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 52: overlapping ranges and nobel numbers

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 53](https://perlweeklychallenge.org/blog/perl-weekly-challenge-053/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


### CORONAVIRUS
The situation in Italy is ugly and to some extent desperate: it has been a few weeks since I'm closed into my house with my son and my wife, we cannot move and cannot go outside. Probably the soldiers will become to monitor the streets very soon this week.
<br/>
<br/>
I cannot go visiting my mum that lives 10 minutes by car from me, and it is not clear when this emergency status will give us a break.
<br/>
<br/>
Therefore, *Perl Weekly Challenge* is not my first thought right now, but at the same time *it is a way to stay in touch with the Perl community and also to get into the same routine that let me think everything is fine*, at least for the time I spend in front of a Perl script!


<a name="task1"></a>
## PWC 53 - Task 1

The first task was about rotating a matrix of integers by a multiple of a square angle.
<br/>
The approach I took was a simple nested loop: every row of the current matrix becomes a column in the rotate matrix. This is true when you want to rotate it clockwise by a square angle, and if you repeat you are able to rotate it by multiple times a square angle.
<br/>
Therefore the core of my application is:


```perl6
 # do a loop for all the rotation
 for 1 .. $rotation / 90 {

     # the matrix is a square matrix, move on the columns
     # to get the rotated row
     for 0 .. @matrix.elems - 1 -> $current-column {
         my @rotated-row;
         for 0 .. @matrix.elems - 1 {
             @rotated-row.push: @matrix[ @matrix.elems - 1 - $_ ][ $current-column ]
         }

         # push the rotated row to the new matrix
         @rotated-matrix.push: [ @rotated-row ];
     }

     # switch the matrix, in the case we need to loop further
     @matrix = @rotated-matrix;
     @rotated-matrix = [];
 }
```

I first compute how many times I have to rotate the matrix, and this is obtained from the user that invokes the application.
The I loop over the `@matrix` elements, knowing that the matrix has to be a square matrix. Then I build a `@rotated-matrix` a row at a time: the `@rotated-row` is built on the current column of the source matrix. Once I've built a rotated row, I push it into what will become the rotated matrix, and then I flip the original `@matrix` with the `@rotated-matrix`, so that I can do another rotation on the same matrix again (if required).
<br/>

The program produces an output similar to:

```perl6
% raku ch-1.p6 180
    Rotation is 180
    Original matrix is 
    | 1 | 2 | 3 | 
    | 4 | 5 | 6 | 
    | 7 | 8 | 9 | 
    Rotated matrix is 
    | 9 | 8 | 7 | 
    | 6 | 5 | 4 | 
    | 3 | 2 | 1 | 
```


<a name="task2"></a>
## PWC 53 - Task 2

The second task involved finding out all fixed length strings made only by vowels with particular rules.
<br/>
At glance I thought a regular expression, or a combination of regular expressions could do the trick, but then I found simpler to implement with a giant *switch* statement.
<br/>
First of all, the user can input the length of the string, and I have to extract all the combinations of such vowels. Let's raku do the hard stuff, and create a huge list of sequences of vowels:

```perl6
my @combinations = @vowels;
for 1 ..^ $size {
    @combinations = @combinations [X] @vowels;
}
```

At the end of the loop, `@combinations` will contain all the vowels mixed up in all possible ways for the specified length.
<br/>
Now it is possible to get every batch of vowels and see if they satisfy the imposed rules. There is a little trick to observe here:
- if a single letter matches, we know for sure it will consume the letter and can move on to the next one;
- the last letter is always ok because it cannot be followed by any other letter.

Therefore, the code looks like:

```perl6
for @combinations -> @letters {
    my $string = @letters.join;
    my $ok = True;

    # test if all but the last letters do match the regular expression
    loop ( my $i = 0; $i < @letters.elems - 1; $i++ ) {
        my $letter = @letters[ $i ];
        $ok = do 
            given $letter {
                when 'a' { $string ~~ / a (e | i) / }
                when 'e' { $string ~~ / ei / }
                when 'i' { $string ~~ / i ( a | e | o | u ) / }
                when 'o' { $string ~~ / o ( a | u ) / }
                when 'u' { $string ~~ /u ( o | e ) / }
        }.so;

        # if not ok, do not continue
        last if ! $ok;
    }

    say "Found { ~@letters }" if $ok;
}
```

<br/>
The program produces the following output:

```perl6
Requested string length is 2 
Found a e
Found a i
Found e i
Found i a
Found i e
Found i o
Found i u
Found o a
Found o u
Found u e
Found u o
```
