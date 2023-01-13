---
layout: post
title:  "Perl Weekly Challenge 103: Chinese Calendar and streaming players"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 103: Chinese Calendar and streaming players

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 103](https://perlweeklychallenge.org/blog/perl-weekly-challenge-103/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




# Seems I missed a couple of PWCs

Yes, and that was on purpose because I was really busy with a professional PostgreSQL training course, so I did not have enough time to get some *extra* stuff.


<a name="task1"></a>
## PWC 103 - Task 1
The first task was about producing a Chinese Calendar script, something that could determine the Chinese year. The hardest part was about understanding how the Chinese Calendar works, and I'm not really sure I got the point but it seems to work.

<br/>
<br/>
```raku
multi sub MAIN( Int $year where { $year > 1900 } ) {
    my @animals = qw/ Rat Ox Tiger Rabbit Dragon Snake Horse Goat Monkey Rooster Dog Pig /;
    my @elements = qw/ Wood Fire Earth Metal Water /;

    say "Year $year is %s %s".sprintf:
        @elements[ ( $year - 4 ) % 10 / 2 ],
        @animals[ ( $year - 4  ) % 12 ];
}

multi sub MAIN() {
    MAIN( DateTime.now.year );
}
```

<br/>

The idea is quite simple: compute a *modulo* on the year and display the animal and the element of the corresponding computation.
<br/>
The only interesting part is that I decided to produce a `multi MAIN` so that you can let the script desume the Chinese year depending on the current date and time. It is interesting to note that the Chinese Calendar starts on February, but I assume this level of detail is not required since the input requested is just the year and not the exact day of the year.

<br/>
### A class-ish implementation

Why not produce a stand-alone class to interact with the Chinese Calendar? That would be something as:

<br/>
<br/>
```raku
class ChineseCalendar {
    has DateTime $!now;
    has Str @!animals;
    has Str @!elements;

    submethod BUILD( :$year = Nil ) {
        @!animals  = qw/ Rat Ox Tiger Rabbit Dragon Snake Horse Goat Monkey Rooster Dog Pig /;
        @!elements = qw/ Wood Fire Earth Metal Water /;
        $!now      = $year ?? DateTime.new: year => $year !! DateTime.now;
    }

    method element() {
        return @!elements[ ( $!now.year - 4 ) % 10 / 2 ];
    }

    method animal() {
        return @!animals[ ( $!now.year - 4 ) % 12 ];
    }
}

```
<br/>
<br/>

Probably there is no need to keep the `DateTime` object, but my idea is that this can help in handling the exact year change related to February.
<br/>
The program therefore changes as:

<br/>
<br/>
```raku
multi sub MAIN( Int $year where { $year > 1900 } ) {
    my $cc = ChineseCalendar.new: year => $year;

    say "Year $year is %s %s".sprintf:
        $cc.element, $cc.animal;
}

multi sub MAIN() {
    MAIN( DateTime.now.year );
}

```
<br/>
<br/>


<a name="task2"></a>
## PWC 103 - Task 2

The second task at glance frightnened me: the explaination was too much longer than the required task, at least this is what I can understand. You have a playback player that loops thru songs, and given the start time, the current time and the list of songs, you have to understand what has been played right now.
<br/>
My idea was to lookup tracks into a `@tracks` array, where each entry is an hash with the duration time and title. Then I iteratively add the length of every song to the start time and see if it reaches the now time, or better I see when the difference between the current time and the start time reaches zero: this means I'm playing the current song.

<br/>
<br/>
```raku
sub MAIN( Int $start-ms where { $start-ms > 0 },
          Int $now-ms where { $now-ms >= $start-ms },
          Str $filename ) {

    # lookup file name and tracks
    my @tracks;
    for $filename.IO.lines {
        my ( $ms, $track ) = .split: ',';
         @tracks.push: { time => $ms,
                         track => $track };
    }

    # try to reach the end of the time
    my $diff = $now-ms - $start-ms;
    my $last-track = 0;
    while ( $diff > 0 ) {
        $last-track = 0;
        for @tracks.kv -> $index, $track {
            $diff -= $track< time >;
            last if ( $diff <= 0 );
            $last-track = $index;
        }
    }

    "Now playing { @tracks[ $last-track ]<track> } ".say;
}
```
<br/>
<br/>

There are different ways this can be improved, for example computing the total amount of the playlist duration and see if that is lower than the difference between times: this can speedup the process by reducing the amount of iterations.
However, for a quick and dirt solution, a nested loop does suffice.
