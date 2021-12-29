---
layout: post
title:  "Perl Weekly Challenge 145: the last challenge of the year!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 145: the last challenge of the year!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 145](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0145/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 1 in plpgsql](#task1pg)
- [Task 2 in plpgsql](#task2pg)





<a name="task1"></a>
## PWC 145 - Task 1
The first task, named *dot product* was really simple to solve in Raku: it was the calculation of the sum of a product across euqlly sized arrays.


<br/>
<br/>
```raku
sub MAIN(  *@n where { @n.elems %% 2 && @n.grep( * ~~ Int ).elems == @n.elems } ) {
    ( [+] ( @n[ 0 .. @n.elems / 2 - 1 ] Z* @n[ @n.elems / 2 .. * - 1 ] ) ).say;
}
 ```
<br/>
<br/>
The idea is simple: all the program input is split into two arrays of equal size, that are multiplied using the zip operator `Z*` that performs the multiplication of elements of the same position. The resulting array is reduced by `[+]` that performs the sum operation. The final result is printed out.



<a name="task2"></a>
## PWC 145 - Task 2

The second task, as very often it happens, was harder to solve: a *palindromic tree* that is the construction of a set of nodes that leave a palindrome string within a given input.

<br/>
<br/>
```raku
sub MAIN( Str $s = 'redivider' ) {

    my @roots;
    my @chars = $s.comb;

    for 0 ..^ @chars.elems -> $current {
        my $current-root = @chars[ $current ];


        for $current + 1 ..^ @chars.elems -> $other {
            next if @chars[ $other ] !~~ $current-root;

            my $string = @chars[ $current .. $other ].join;

            if ( $string ~~ $string.flip ) {
                @roots.push: [ $current-root, $string ];
                last;
            }
        }

        @roots.push: [ $current-root, '' ] if ! @roots.grep({  $_[ 0 ] ~~ $current-root } );
    }

    "$_[0] = $_[1]".say for @roots;
}


```
<br/>
<br/>

I used a nested loop: in the outer loop I take a `$current-root` possible solution, and then try to find out the first palindrome string I can find (with the inner loop). Once I found the palindrome string, I then push an array with the root letter and the resulting string.

Last, I do print all the solutions I found.

<a name="task1pg"></a>
## PWC 145 - Task 1 in PostgreSQL

The first task could be achived with a `FOR` loop over the array size (the array are supposed to be equally sized) and cumulatively compute the sum of the products.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
f_dot_product( a int[], b int[] )
RETURNS int
AS
$CODE$
DECLARE
        i int;
        total int := 0;
BEGIN
        FOR i IN 1 .. array_length( a, 1 ) LOOP
            total := total + a[ i ] * b[ i ];
        END LOOP;

        RETURN total;
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>

<a name="task2pg"></a>
## PWC 145 - Task 2 in PostgreSQL

This is a clear re-implementation of what I did in Raku:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_eertree( s text )
RETURNS TABLE( current_root char, string text )
AS $CODE$
DECLARE
        current int;
        other   int;
        other_root   char;
BEGIN

        FOR current IN 1 .. length( s ) LOOP
           current_root := substring( s FROM current FOR  1 );

           FOR other IN current + 1 .. length( s ) LOOP
               other_root := substring( s FROM other FOR 1 );
               IF other_root <> current_root THEN
                  CONTINUE;
               END IF;

               string := substring( s, current, other - current + 1 );

               IF string = reverse( string ) THEN
                  RETURN NEXT;
               END IF;

           END LOOP;
        END LOOP;

        RETURN;

END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>

A particular aspect of this function is that it returns a `TABLE`, that is something that has columns and rows. Columns declared in the output table become lexically visible variables, so that it does suffice to assign to them a value and issue a `RETURN NEXT` to append all of them to the result set.
