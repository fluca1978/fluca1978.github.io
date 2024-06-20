---
layout: post
title:  "Perl Weekly Challenge 274: not understood"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 274: not understood

This post presents my solutions to the [Perl Weekly Challenge 274](https://perlweeklychallenge.org/blog/perl-weekly-challenge-274/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 274 - Task 1 - Raku](#task1)
- [PWC 274 - Task 2 - Raku](#task2)
- [PWC 274 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 274 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 274 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 274 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 274 - Task 1 in PL/Java](#task1pljava)
- [PWC 274 - Task 2 in PL/Java](#task2pljava)
- [PWC 274 - Task 1 in Python](#task1python)
- [PWC 274 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 274 - Task 1 - Raku Implementation

The first task was about the transliteration of a sentence in *goat latin*, with the following rules:
- if a word begins with a vowel , append "ma" to the end of the word;
- if a word begins with consonant i.e. not a vowel, remove first letter and append it to the end then add "ma";
- add letter "a" to the end of first word in the sentence, "aa" to the second word, etc etc.

<br/>
<br/>
```raku
sub MAIN( Str $sentence ) {
    my $suffix = 'a';
    my @words;
    for $sentence.split( / \s+ / ) -> $word is copy {
		$word ~~ / ^ ( <[a..zA..Z]> ) .* $ /;
		my $initial = $/[ 0 ].Str;
		if ( $initial.lc !~~ / <[aeiou]> / ) {
		    $word = $word.comb[ 1 .. * ].join ~ $initial;
		}

		$word ~= 'ma';
		$word ~= $suffix;
		$suffix ~= 'a';
		@words.push: $word;
    }

    @words.join( ' ' ).say;

}

```
<br/>
<br/>


The idea is quite simple: first I `split` the `$sentence` into words, then capture the first letter with a regular expression.
If the first letter is not a vowel, I reconstruct the word taking only the second character to the end, and append the captured letter.
Then I append `ma` to the end and the suffix, extending the `$suffix` at every run.



<a name="task2"></a>
## PWC 274 - Task 2 - Raku Implementation


This task was really complicated for me to understand, and in fact the final solution provides much more results than it should, but it is almost fine for me.
The task was providing a set of *bus routes* with the starting time (after the hour), the interval for the next bus and the time required to arrive, all expressed in minutes.
The task was requiring to understand when there is possibility to skip the *main* bus route in favor of another one that starts a little after and arrives sooner than the main one.

<br/>
<br/>
```raku
sub MAIN() {

    my @routes = [ [12, 11, 41], [15, 5, 35] ];

    my @times;
    my $route = 0;

    my %timetable;

    my $preferred = False;
    for @routes -> $current_route {

		my ( $interval, $start_at, $time ) = $current_route;

		my $arrive_at = $start_at + $time;
		my $previous_bus_start_at = 0;

		while ( $arrive_at < 120 ) {
		    for 0 .. 59 -> $minute {
						last if $minute > $start_at;
						next if $minute < $previous_bus_start_at;
						%timetable{ $minute }.push: { start => $start_at, arrive => $arrive_at, route => $current_route.Str, preferred => $preferred };
		    }

		    $previous_bus_start_at = $start_at;
		    $start_at += $interval;
		    $arrive_at = $start_at + $time;
		}

		$preferred = True;
    }

    my @skip;
    for %timetable.keys.sort( * <=> * ) -> $minute {
		my @bus = %timetable{ $minute }.Array;
		next if ( @bus.elems == 1 && ! @bus[ 0 ]<preferred> );

		@skip.push: $minute if ( @bus.sort( { $^a<arrive> <=> $^b<arrive> } )[ 0 ]<preferred> );
    }


    @skip.say;



}

```
<br/>
<br/>


My implementation works as follows.
First I iterate over all possible minutes in an hour, and for evrey minutes I take the "closest" timetable for every bus, placing them in a list into `%timetable`. The idea is that `%timetable` will provide you, for every minute in an hour, when you could take a bus. For example, at minute `9` you could take route one at minute `11` or route 2 at minute `10`.
Every bus route is stored into the `%timetable` with the details about the `start` time, the `arrive` time (in absolute minutes, e.g., `70` means `1 hour and 10 minutes`) and if it is a *preferred* bus, with the first one route being not the preferred one.

Then I iterate over all the minutes again, and check which `@bus` are availables for that minute or shortly after. If the bus that arrives sooner is a preferred one, it means I can choose a better bus on such minute or shortly after, and hence the minute can be *skipped* meaning I can wait for a preferred bus that will get me sooner to the destination.

**However, this is not the implementation that provides the right solution, it does provide a larger set of solution values!* This is due to the fact that I've not understood fully the problem specification.

Hence, I will not implement it in other languages.
