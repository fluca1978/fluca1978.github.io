---
layout: post
title:  "Perl Weekly Challenge 184: mangling strings"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 184: mangling strings

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 184](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0184/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)


Last, the solutions in PostgreSQL PL/PgSQL:

<br/>
- [Task 1 in PostgreSQL Pl/PgSQL](#task1plpgsql)
- [Task 2 in PostgreSQL Pl/PgSQL](#task2plpgsql)

<a name="task1"></a>
## PWC 184 - Task 1

Write an application that accepts a set of strings, all beginning with two letters followed by four digits. Replace the letters with a two digit counter starting from 1.

<br/>
<br/>
```raku
sub MAIN( *@strings where { @strings.grep( $_ ~~ / ^ <[a..z]> ** 2 \d ** 4 $/ ).elems
                                             == @strings.elems } ) {
    my $counter = 0;
    my @ordered-strings = @strings.map: {
        my $s = $_;
        $s ~~ s/ ^ ( <[a..z]> ** 2 ) /{ "%02d".sprintf( $counter++ ) }/;
        $s; };
    @ordered-strings.join( "\n" ).say;

}
 ```
<br/>
<br/>

The `where` checks that user has specified only strings in the right format. Then, thru `map` I replace the initial letters with the result of a code expression, a `sprintf`.


<a name="task2"></a>
## PWC 184 - Task 2
The user can provide to the script a set of strings where every character is separated by the following one by a space.
The script has to keep track of all the digits and all the letters, and present them as separaterd couple of arra

<br/>
<br/>
```raku
sub MAIN( *@strings ) {
    my @numbers;
    my @letters;

    for @strings -> $current-string {
        my ( @n, @l );
        for $current-string.comb {
            @n.push: $_ if ( $_ ~~ / \d / );
            @l.push: $_ if ( $_ ~~ / <[a..z]> / );
        }

    @numbers.push: [ @n ] if ( @n );
    @letters.push: [ @l ] if ( @l );
    }

    @numbers.join( ", " ).say;
    @letters.join( ", " ).say;
}

```
<br/>
<br/>


<a name="task1plperl"></a>
## PWC 184 - Task 1 in PostgreSQL PL/Perl

essentially, the same solution from the Raku implementation translated into Perl:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc184.task1_plperl( text[] )
RETURNS SETOF text
AS $CODE$

my $counter = 0;
for my $current_string ( @{$_[0]} ) {
    next if $current_string !~ / ^ [a-z]{2} \d{4} $ /ix;
    $counter = sprintf "%02d", $counter;
    $current_string =~ s/ ^ [a-z]{2} /$counter/xi;
    $counter++;
    return_next( $current_string );
}

return undef();
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 184 - Task 2 in PostgreSQL PL/Perl

Similar to the Perl implementation, except that I return a `TABLE` and therefore an hash reference.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc184.task2_plperl( text[] )
RETURNS TABLE( n text, l text )
AS $CODE$


my @numbers;
my @letters;

for my $current_string ( @{ $_[0] } ) {
    my @parts = split //, $current_string;

    for ( @parts ) {
        push @numbers, $_ if ( $_ =~ /\d/ );
        push @letters, $_ if ( $_ =~ /[a-z]/i );
    }
}

return_next( { n => join( ',', @numbers ),
          l => join(',', @letters ) } );
return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 184 - Task 1 in PostgreSQL PL/PgSQL

The same approach in the previous solutions, note the usage of `FOR IN ARRAY` to iterate over all the input strings.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc184.task1_plpgsql( strings text[] )
RETURNS SETOF text
AS $CODE$
DECLARE
        current_string text;
        c int := 0;
        pref text;
BEGIN
     FOREACH current_string IN ARRAY strings LOOP
       IF c < 10 THEN
          pref := '0' || c;
       ELSE
         pref := c::text;
      END IF;
       RETURN NEXT regexp_replace( current_string,
                                   '^[a-z]{2}',
                                   pref );
       c := c + 1;
     END LOOP;
     RETURN;
END


$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

One problem is that PostgreSQL does not provide a `sprintf` like function, so I used an intermediate string `pref` as `prefix` to place in front of the string, and in the case the value is less than `10` I place a `0` in front of the number.


<a name="task2plpgsql"></a>
## PWC 184 - Task 2 in PostgreSQL PL/PgSQL

Same approach of the previous solutions.
Please note the usage of regular expressions by means of the `~` operator.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc184.task2_plpgsql( strings text[])
RETURNS TABLE (n text, l text)
AS $CODE$
DECLARE
        current_string text;
        current_thing text;
BEGIN
    n := null;
    l := null;
    FOREACH current_string IN ARRAY strings LOOP
       FOREACH current_thing IN ARRAY regexp_split_to_array( current_string, '' ) LOOP
         -- since '\w' gets also numbers
         -- the test is performed only if it is not
         -- a number
         IF current_thing ~ '\d' THEN
            IF n IS NULL THEN
               n := current_thing::text;
            ELSE
              n := n || ',' || current_thing;
            END IF;
         ELSEIF current_thing ~ '\w' THEN
            IF l IS NULL THEN
               l := current_thing::text;
            ELSE
              l := l || ',' || current_thing;
            END IF;
         END IF;
 END LOOP;
    END LOOP;

    RETURN NEXT;
    RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
