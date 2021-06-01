---
layout: post
title:  "Perl Weekly Challenge 114: palindrome and '1's numbers"
author: Luca Ferrari
tags:
- perlweeklychallenge
- raku
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 114: palindrome and '1's numbers

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
## PWC 114 - Task 1
The first task was abount finding the next palindrome number greater thana  given one.
<br/>
It is a quite simple task to me: a number is palindrome if its `flip`ped string representation is the same, so it all means I have to loop over numbers bigger than the current one and see if it is a palindrome.

<br/>
<br/>
```raku
sub MAIN( Int $N where { $N > 10 } ) {
    say $_ and exit  if $_ == $_.Str.flip.Int for $N ^.. Inf;
}

```
<br/>
<br/>

Therefore I loop from `$N` (exlcuded), i.e., the given number, to the infinity. For every number, `$_`, I compare it to its flipped version, and the if the numbers are the same, then I do print the value and exit.
<br/>
Please note the lower precedence `and` in the `say $_ and exit` that avoid `exit` to take precedence on the whole thing and terminate the program without printing anything.


<a name="task2"></a>
## PWC 114 - Task 2
The second task was a kind of one liner too: find out a number that has the same amount of `1` digits in the binary representation, assuming it is bigger than a give value.

<br/>
<br/>
```raku
sub MAIN( Int $N where { $N > 0 } ) {
    say $_ and exit if ( $N.base( 2 ).split( '' ).grep( '1' ).elems 
                              == $_.base( 2 ).split( '' ).grep( '1' ).elems ) 
                                  for $N ^.. Inf;

}
```
<br/>
<br/>

Again, I do loop from the given number to the infinity, and for every value I check if the number, converted into binary thru `.base( 2 )`, reduced to an array of digits by means of `split( '' )`, extracted to its only `1` digits by means of `grep( '1' )` has the same number of elements of the current number.
<br/>
In the case, I do print the value and terminate the script.
<br/>
Of course, this is a concise approach, but is not optimized, since it computes over and over again the number of `1` digits of the given number `$N`.
