---
layout: post
title:  "Perl Weekly Challenge 48: Survivors and Palindrome Dates"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 48: Survivors and Palindrome Dates

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 48](https://perlweeklychallenge.org/blog/perl-weekly-challenge-048/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<a name="task1"></a>
## PWC 48 - Task 1

The first task required to find out a survivor in a kind of *Philosopher Dining Problem*: there are 50 people sitting in a circle, one of them has a sword and kills the next one passing the sword to the next-to-the-just-killed one. This goes on until a single one survives.
<br/>
I decided to implement the people as a list where a `True` value indicates that that person has the sword, a `False` that the person does not have a sword and a `Nil` that the person has been killed.
<br/>
The only problem I found, is to rotate over the array, so I defined a specific method to find the next alive person given a specific index:

```perl6
sub next-alive( @people, $current-person ) {
    my $next = $current-person;

    loop {
        $next++;
        $next = $next >= @people.elems ?? $next % @people.elems !! $next;
        return $next if @people[ $next ].defined;
    }
}
```

I did not find any more elegant approach to loop over the array.
<br/>
Having that, it becomes quite simple to iterate over the people and  loop until only one remains:


```perl6
while ( @people.grep( *.defined ) > 1 ) {
    # find out who has the sword
    my $killer      = @people.first: *.so, :k;
    # then find out the next person to kill
    my $killed      = next-alive( @people, $killer );
    @people[ $killed ] = Nil;  # killed!
    @people[ $killer ] = False; # pass the sword
    # now get the next person that will hold the sword
    my $next-killer = next-alive( @people, $killed );
    @people[ $next-killer ] = True; # the next killer
}
```

While there is only one not-`Nil` value in the array, I select the first people with the sword and then the next one alive near the killer, and finally the next one to the killed person. Then it is just a metter of changing the values.
<br/>
The final result is that running the program produces:

```perl6
% perl6 ch-1.p6
The person who survives is 37
```



<a name="task2"></a>
## PWC 48 - Task 2

The second task was simpler to me: produce palindrome dates.
<br/>
I decided to implement this using `Date` and placing the `formatter` property, then I looped one day at a time and print out the date if its stringyfied version is the same as its reversed (i.e., `flip`) stringyfied value:

```perl6
    my $current-date = Date.new( :year( $year-start ),
                                 :day(1),
                                 :month(1),
                                 formatter => { sprintf( "%02d%02d%04d", .month, .day, .year ) } );
    my $end-date     = Date.new( :year( $year-end ), :day(31), :month(12) );

    for 1 .. $end-date - $current-date {
        $current-date += 1;  # add one day at a time
        # print the date if its string representation is the same as
        # the flipped string representation
        "Palindrome: $current-date".say if $current-date.Str ~~ $current-date.Str.flip;
    }
```

However, after some thoughts (night was useful!), I realized that such a *brute force* way was too much expensive. Knowing that the date must be palindrome, **it means that given an year we already know what the month and the day will be, and therefore the only thing to do is to check if such a date is good or not**.
[I then re-implemeted the program as follows](https://github.com/fluca1978/fluca1978-coding-bits/commit/d308f22c96f76f32ae92cbb530b19dfa1d3bc0e5){:target="_blank"}:


```perl6
    for $year-start .. $year-end {
        $_ ~~ / ^ $<day>=\d ** 2 $<month>=\d ** 2 $ /;
        my $month = $/<month>.flip;
        my $day   = $/<day>.flip;
        next if  $month > 12 || $month == 0;
        next if $day > 31 || $day == 0;
        "Palindrome date %02d%02d%04d".sprintf( $month, $day, $_ ).say if try Date.new( :year( $_),
                                                                                    :month( $month),
                                                                                    :day( $day ) );
    }
```

First, I extract the parts that will become the month and the day from the year string. Then I flip them because I need them reversed to make a palindrome date. It is simple enough to exclude obvious *out of range* values, like a month greater than `12` or a day greater than `31`. This however does not suffice to give me a valida te, so I try to construct a `Date` object, and if I succeed I then have a valid palindrome date.
<br/>
Please note the use of `try` within the `if`: `Date` will throw an exception if the construction fails, so I don't want my loop to break if I'm trying a stupid date with out of range values.

