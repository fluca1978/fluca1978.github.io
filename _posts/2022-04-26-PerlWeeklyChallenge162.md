---
layout: post
title:  "Perl Weekly Challenge 162: too much complicated?"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 162: too much complicated?

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 162](https://perlweeklychallenge.org/blog/perl-weekly-challenge-162/){:target="_blank"}.

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
## PWC 162 - Task 1

The first task was about finding out the check digit of an ISBN-13 code. That was not difficult, once the alghoritm is clear enough:



<br/>
<br/>
```raku
sub MAIN( Str $isbn where { $isbn.chars == 12 && $isbn.comb.grep( * ~~ / <[0 .. 9]> / ).elems == $isbn.chars } ) {
    my $position = 0;
    say 10 - ( $isbn.comb.map( { $_ * ( ++$position %% 2 ?? 3 !! 1 ) } ).sum % 10 );
}


 ```
<br/>
<br/>

The trick is to sum all the first 12 digits with a multiplier that depends on the position of the digit, than you can compute the remainder of the modulo 10 operation.


<a name="task2"></a>
## PWC 162 - Task 2

*This was too much complicated, at least for me!*
<br/>
Implementing a cipher, even if as simple as *FairPlay* can be, was a very long and somehow boring job.
<br/>
The cipher words on a passhprase table: the passphrase is translated into a table, a square table. Every letter of the incoming message is then translated to another letter depending on its position on the table. In particular the clear message is split into couple of letters, each of them is then positioned in the table and if they form a rectangle, or a column or a row, a translation rule to extract other two letters from the table is applied.
<br/>
Decyption works pretty much the sme, with some more rules because of meaningless letters (e.g., `X`) in the incoming message.
<br/>
I implemented it in a class way:
- `Passprhase` accepts a text string and builds up the tablò according to the padding rules;
- `Cypher` builds a `Passprhase` and provides method to `encrypt` and `decrypt` the text that was used to initialize it.

<br/>
Therefore, the `MAIN` program looks as simple as:

<br/>
<br/>
```raku
sub MAIN() {
    my Cypher $cypher = Cypher.new: text => 'hide the gold in the tree stump',
                        passphrase => 'playfair example';

    "Message\n{ $cypher.text }\n with passphrase table\n{ $cypher.pass.grid.map( *.join ).join }\n\n\t => { $cypher.encrypt }".say;



    $cypher = Cypher.new: passphrase => "perl and raku",
              text => "siderwrdulfipaarkcrw";

    "Encrypted message \n{ $cypher.text }\n with passphrase table\n { $cypher.pass.grid.map( *.join ).join }\n\n\t => { $cypher.decrypt }".say;
}
}

```
<br/>
<br/>

The `Passphrase` class is the simplest one, since it takes a string of text and builds a matrix `5x5` used to encrypt or decrypt a message:


<br/>
<br/>

``` raku
class Passphrase {
    has Str $!passphrase;
    has @.grid;

    method BUILD( Str :$passphrase ) {
        $!passphrase = $passphrase;

        # build the grid
        my @chars =  $!passphrase.uc.comb;
        @chars.push: $_ if $_ !~~ /  J / for 'A' .. 'Z';

        my @current-row;
        my %seen;
        for @chars {
            next if $_ ~~ / ' ' /;
            @current-row.push( $_ ) and %seen{ $_ }++ if ! %seen{ $_ };
            @!grid.push: [ @current-row.reverse ] and @current-row = ()  if @current-row.elems == 5;
            last if @!grid.elems == 5;
        }
    }
}

```
<br/>
<br/>

The only trick here is that a letter can appear twice in the table and the `J` is equivalent to `I`.

<br/>
Then it comes `Cypher`, that first of all constructs its `Passphrase` and split the incoming message into couple of letters according to some rules, like space removal, duplicate substitution, and so on:


<br/>
<br/>

``` raku
class Cypher {
    has Passphrase $.pass;
    has Str $.text;
    has @.sequences;

    method BUILD( Str :$text, Str :$passphrase ) {
        $!pass = Passphrase.new: passphrase => $passphrase;
        $!text = $text.uc;
        $!text ~~ s:g/\s+//;
        $!text ~= 'X' while ( $!text.chars !%% 2 );

        my @chars = $!text.comb;

        loop ( my $i = 0; $i < @chars.elems - 1; $i++ ) {
            my ( $a, $b ) = @chars[ $i ], @chars[ ++$i ];
            @!sequences.push: [ $a, $b ] if $a !~~ $b;
            @!sequences.push( [ $a, 'X' ] ) and $i-- if $a ~~ $b;
        }

        $!text = $text.uc;
    }
...
```
<br/>
<br/>

The `@!sequences` is the array of couple of letters to process. The `do-sequences` method does the machinery: it inspects every couple of letters and find them in the table as set of coordinates. Then it inspects the coordinates to find out if they are layed out as a rectangular, a column or a row. Depending on the layout, the letters are translated to toher letters, also depending on the *direction*, defined for encryption and decryption:


<br/>
<br/>

``` raku
    method do-sequences( :$encode = True ){
        my @translated-coordinates;

        for @!sequences {
            my @coordinates = self!find-coordinates( $_ );
            my @new-coordinates;
            #say $_.join(',') ~ " found at " ~ @coordinates.raku;


            my $is-rectangle = @coordinates[ 0 ]< row > != @coordinates[ 1 ]< row >
            && @coordinates[ 0 ]< col > != @coordinates[ 1 ]< col >;

            my $is-column = @coordinates[ 0 ]< row > != @coordinates[ 1 ]< row >
            && @coordinates[ 0 ]< col > == @coordinates[ 1 ]< col >;

            my $is-row = @coordinates[ 0 ]< row > == @coordinates[ 1 ]< row >
            && @coordinates[ 0 ]< col > != @coordinates[ 1 ]< col >;

            if $is-rectangle {
                @new-coordinates.push: %(
                    row =>  @coordinates[ 0 ]< row >,
                    col => @coordinates[ 1 ]< col > );

                @new-coordinates.push: %(
                    row =>  @coordinates[ 1 ]< row >,
                    col => @coordinates[ 0 ]< col > );
            }
            elsif $is-column {
                @new-coordinates.push: %(
                    row =>  ( @coordinates[ 0 ]< row >
                              + ( $encode ?? 1 !! -1 ) )
                             % $!pass.grid.elems,
                    col => @coordinates[ 0 ]< col > );

                @new-coordinates.push: %(
                    row =>  ( @coordinates[ 1 ]< row > + ( $encode ?? 1 !! -1 ) )
                            % $!pass.grid.elems,
                    col => @coordinates[ 1 ]< col > );
            }
            elsif $is-row {
                @new-coordinates.push: %(
                    row => @coordinates[ 0 ]< row >,
                    col => ( @coordinates[ 0 ]< col > + 1 )
                           % $!pass.grid[ 0 ].elems
                );

                 @new-coordinates.push: %(
                     row => @coordinates[ 1 ]< row >,
                     col => ( @coordinates[ 1 ]< col > + 1 ) % $!pass.grid[ 1 ].elems
                );
            }

            @translated-coordinates.push: ( @new-coordinates );

#            say "Block " ~ self!chars-at-coordinates( @coordinates ) ~ " substitued with " ~ self!chars-at-coordinates( @new-coordinates);
        }

        return @translated-coordinates;
    }

```
<br/>
<br/>

There are a couple of utility methods used to find out a block of two letters in the table and process all the couples to get all the translated coordinates and letters:


<br/>
<br/>

``` raku
    method !chars-at-coordinates( @coord ) {
        return if ! @coord;
        my @chars;
        for @coord -> $point {
            next if ! $point;
            @chars.push: $!pass.grid[ $point< row > ][ $point< col > ];
        }

        return @chars;
    }

    method !find-chars( @coords ) {
        my @chars;
        for @coords -> @coord {
            @chars.push: self!chars-at-coordinates( @coord );
        }

        return @chars;
    }
    method !find-coordinates( @sequence ) {
        my @coordinates;

        for 0 ..^ @sequence.elems {
            my $needle = @sequence[ $_ ];
            my $found = False;

            for 0 ..^ $!pass.grid.elems -> $row {
                for 0 ..^ $!pass.grid[ $row ].elems -> $col {
                    if $needle ~~ $!pass.grid[ $row ][ $col ] {
                        # found!
                        @coordinates.push: %( row => $row, col => $col );
                        $found = True;
                    }

                    last if $found;
                }

                last if $found;
            }
        }

        return @coordinates;
    }

```
<br/>
<br/>

Last come the `encrypt` and `decrypt` methods, that essentially call `do-sequences` and extract the resulting letters from the translated coordinates:


<br/>
<br/>

``` raku
    method encrypt() {
        self!find-chars( self.do-sequences() ).map( *.join ).join;
    }

    method decrypt() {
        my @chars = self!find-chars( self.do-sequences( encode => False ) );
        my $text;
        for @chars -> @pair {
            next if ! @pair;
            @pair[ 1 ] = @pair[ 0 ] if @pair[ 1 ] ~~ / X /;
            $text ~= @pair.join;
        }

        return $text;
    }
```

<a name="task1plperl"></a>
## PWC 162 - Task 1 in PostgreSQL PL/Perl


A quite simple translation of the Raku implementation:

<br/>
<br/>

``` perl
CREATE OR REPLACE FUNCTION
pwc162.isbn13_check_digit( text )
RETURNS int
AS $CODE$
   my ( $isbn ) = @_;
   my $sum = 0;
   my $position = 0;

   $sum += $_ for ( map { $_ * ( ++$position % 2 == 0 ? 3 : 1 ) }  split( //, $isbn ) );

   return 10 - ( $sum % 10 );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


<a name="task2plperl"></a>
## PWC 162 - Task 2 in PostgreSQL Pl/Perl

I think it is possible, but I have no time (and at the moment, will) to implement it!

## PWC 162 - Task 1 in PostgreSQL Pl/PgSQL

I split the incoming ISBN code to a table, loop over it thru a `FOR` cursor and compute the sum of the digits. After all, it is simple, but too much verbose because of the constructs (un)available in Pl/PgSQL:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc162.isbn13_check_digit_plpgsql( isbn text )
RETURNS int
AS $CODE$
DECLARE
    sum int := 0;
    pos int := 0;
    i   record;
BEGIN
    FOR i IN SELECT v FROM regexp_split_to_table( isbn, '' ) v LOOP
        pos := pos + 1;
        IF pos % 2 = 0 THEN
           sum := sum + i.v::int * 3;
        ELSE
           sum := sum + i.v::int;
        END IF;
    END LOOP;

    RETURN 10 - ( sum % 10 );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


<a name="task2plpgsql"></a>
## PWC 162 - Task 2 in PostgreSQL Pl/PgSQL

Probably it could be built with some CTE and functions, but again, I have no time this week to try it out!
