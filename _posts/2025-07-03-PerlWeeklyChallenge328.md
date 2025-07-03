---
layout: post
title:  "Perl Weekly Challenge 328: regexs everywhere!"
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

# Perl Weekly Challenge 328: regexs everywhere!

This post presents my solutions to the [Perl Weekly Challenge 328](https://perlweeklychallenge.org/blog/perl-weekly-challenge-328/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 328 - Task 1 - Raku](#task1)
- [PWC 328 - Task 2 - Raku](#task2)
- [PWC 328 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 328 - Task 2 in PostgreSQL PL/Perl](#task2plperl)

# Raku Implementations

<a name="task1"></a>
## PWC 328 - Task 1 - Raku Implementation


The first task was to substitute all question marks in a word with a letter that has to be randomly chosen so that it is not the same as both the preceding and following one.

This can be solved in a single line, even if long!


<br/>
<br/>
```raku
sub MAIN( Str $string  where { $string ~~ / '?'+ / }  ) {
    $string.subst( / $<left>=(<[a .. z]>) <[?]> $<right>=(<[a .. z]>)? /,
							    { $<left> ~ ('a'..'z').grep( { $_ ne $<left> && $_ ne ( $<right> // '' ) } ).first ~ ( $<right> // '' ) },
							    :g )
    .say;

}

```
<br/>
<br/>

I do an in place `subst`itution assigning to `$<left>` and `$<right>` to the letters around the question mark, so that I grep from the list of letters the first one that is not the same as the previous two. I concatenate such letter in the middle of `$<left>` and `$<right>`.
Last I print out the final result.



<a name="task2"></a>
## PWC 328 - Task 2 - Raku Implementation

The second task was about removing duplicated letters in a row, igonring the case.


<br/>
<br/>
```raku
sub MAIN( Str $string is copy where { $string ~~ / ^ <[a..zA..Z]>+ $ / } ) {

    my @chars = $string.lc.comb;
    my @result;
    my $index = 0;

    while ( $index < @chars.elems ) {
		@result.push: @chars[ $index ] and last if ( $index == @chars.elems - 1 );

		if ( @chars[ $index ] ne @chars[ $index + 1 ] ) {
		    @result.push: @chars[ $index ];
		    $index++;
		}
		else {
		    $index += 2;
		}
    }

    @result.join.say;
}

```
<br/>
<br/>

This is too much difficult for a regex to be designed by me!
So I iterate over the lowercase characters and look if the current pair are the same, in such case I go ahead two chars, otherwise I push the current one in the `@result` array that is, at last, joined with nothing.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 328 - Task 1 - PL/Perl Implementation

Using regular expressions, it is quite simple to construct the substitution and apply it one question mark at a time.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc328.task1_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;

   while ( $string =~ / ([a-z]) [?] ([a-z])? /x ) {
   	 my $question_mark = ( grep { $_ ne $1 && $_ ne $2 }  ( 'a' .. 'z' ) )[ 0 ];
	 my $replacement = $1 . $question_mark . $2;
   	 $string =~ s/([a-z]) [?] ([a-z])?/$replacement/x;
   }

   return $string;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 328 - Task 2 - PL/Perl Implementation

This is the same implementation as Raku.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc328.task2_plperl( text )
RETURNS text
AS $CODE$

   my ( $string ) = @_;

   $string = lc $string;
   my @chars = split //, $string;
   my @result;

   my $index = 0;
   for ( 0 .. $#chars ) {
       if ( $chars[ $index ] eq $chars[ $index + 1 ] ) {
       	  $index += 2;
       }
       else {
       	    push @result, $chars[ $index ];
	    $index++;
       }
   }

   return join( '', @result );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

