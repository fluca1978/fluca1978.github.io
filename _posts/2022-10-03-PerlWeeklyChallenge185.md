---
layout: post
title:  "Perl Weekly Challenge 185: string substitutions"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 185: string substitutions

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 185](https://perlweeklychallenge.org/blog/perl-weekly-challenge-185/){:target="_blank"}.

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
## PWC 185 - Task 1

Given a mac address in the form `aaaa.aaaa.aaaa` provide the well know form `aa:aa:aa:aa:aa:`.
It is quite simple to do this:

<br/>
<br/>
```raku
sub MAIN( Str $input-mac-address
          where { $input-mac-address ~~ / ^ ( <[0..9a..f]> ** 4 <[.]> ) ** 2  <[0..9a..f]> ** 4 $ / } ) {

    my $output-mac-address = $input-mac-address.subst( /<[.]>/, '', :g );
    $output-mac-address.comb( :spit-empty ).rotor( 2 ).join( ':' ).subst( /\s+/, '', :g ).say;

}

 ```
<br/>
<br/>

The regular expression in the parameter list ensures that the input is what I expect to be.
Then I remove the dots within the mac address, obtaining a string-only representation, that then I split and recombine in groups of `2` letters via `rotor`, append a `:` at the array join operation and remove all the spaces.


<a name="task2"></a>
## PWC 185 - Task 2
Substitute the first four digits or letters found in a set of random strings.

<br/>
<br/>
```raku
sub MAIN( *@codes ) {
    my @output-codes;
    for @codes -> $current-code {
        my @current-output;
        my $counter = 4;
        for $current-code.comb -> $current-char is copy {
            $current-char = 'x'
                   and $counter--
                      if $current-char ~~ / <[0..9a..z]> / && $counter > 0;
            @current-output.push: $current-char;
        }

        @output-codes.push: @current-output.join;
    }

    @output-codes.join( "\n" ).say;
}

```
<br/>
<br/>

The idea is to to split every string into its array of chars, and then keep track with a `$counter` of how many I need to substitute. I recombine the chars into the `@current-output` array that is then printed as a string.

<a name="task1plperl"></a>
## PWC 185 - Task 1 in PostgreSQL PL/Perl

A very similar implementation to the Raku one.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc185.task1_plperl( text )
RETURNS text
AS $CODE$
my ( $input ) = @_;

$input =~ s/\./g/;

my ( $counter, $output ) = ( 1, '' );
for ( split( //, $input ) ) {
    $output .= $_;
    $output .= ':' if $counter % 2 == 0;
    $counter++;
}

return $output;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

This time I keep a `$counter` that holds how many chars I've already placed in the final mac address string, and each time I get a multiple of `2` I place also a `:`.


<a name="task2plperl"></a>
## PWC 185 - Task 2 in PostgreSQL PL/Perl

Similar to the Raku implementation, but accepts a single string as input.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc185.task2_plperl( text )
RETURNS text
AS $CODE$
my ( $input ) = @_;
my @output;
my $counter = 4;

for ( split( //, $input ) ) {
    push @output, 'x'
        and $counter--
        and next if ( /[a-z0-9]/i ) and $counter > 0;
    push @output, $_;
}

return join( '', @output );
$CODE$
LANGUAGE plperl;


```
<br/>
<br/>


<a name="task1plpgsql"></a>
## PWC 185 - Task 1 in PostgreSQL PL/PgSQL

A reimplementation of the PL/Perl approach.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc185.task1_plpgsql( mac_input text )
RETURNS text
AS $CODE$
DECLARE
        c int := 1;
        d char;
        mac_output text := '';
BEGIN
        mac_input := replace( mac_input, '.', '' );
        FOREACH d IN ARRAY regexp_split_to_array( mac_input, '' ) LOOP
                mac_output := mac_output || d;
                IF c % 2 = 0 THEN
                   mac_output := mac_output || ':';
                END IF;
                c := c + 1;
        END LOOP;

        RETURN mac_output;
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>




<a name="task2plpgsql"></a>
## PWC 185 - Task 2 in PostgreSQL PL/PgSQL

Similar approach than the PL/Perl one.
I use the `~` regular expression match operator to quickly understand if the character I'm analysing is a digit, a letter or none of the above.

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc185.task2_plpgsql( code text )
RETURNS text
AS $CODE$
DECLARE
        how_many int := 4;
        d char;
        code_out text := '';
BEGIN
        FOREACH d IN ARRAY regexp_split_to_array( code, '' ) LOOP
                IF how_many = 0 OR d !~ '\w'  THEN
                   code_out := code_out || d;
                   CONTINUE;
               ELSEIF how_many > 0 AND d ~ '\w' THEN
                    code_out := code_out || 'x';
                    how_many := how_many - 1;

               ELSE
                    code_out := code_out || d;
               END IF;
        END LOOP;

RETURN code_out;
END
$CODE$
LANGUAGE plpgsql;


```
<br/>
<br/>
