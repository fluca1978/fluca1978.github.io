---
layout: post
title:  "Perl Weekly Challenge 182: max and containing paths
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 182: max and containing paths

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 182](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0182/){:target="_blank"}.

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
## PWC 182 - Task 1

Quite easy: given an input list of numbers, find out the position of the greates one within the list.
I decided to implement it via an hash, that is keyed by the value and contains the corresponding position of the key within the list. Yes, it is an overkilling approach, but allows me to avoid the boring looping:



<br/>
<br/>
```raku
sub MAIN( *@n ) {
    my %h;
    %h{ @n[ $_ ] } = $_ for 0 ..^ @n.elems;
    "{ %h{ $_ } }".say given %h.keys.max;
}

 ```
<br/>
<br/>



<a name="task2"></a>
## PWC 182 - Task 2

Given a set of absolute paths, find out the most common deep container.
<br/>
In the beginning I thought about using an hash, but I need to keep sorting and ensure that every diretory on the path has the same position in all other paths. I consider no files are involved here.

<br/>
<br/>
```raku
sub MAIN( *@paths ) {
    my @common-path;


    my @path-pieces;
    @path-pieces.push: [ .split( '/', :skip-empty ) ] for @paths;
    my $min-pieces = @path-pieces[0].elems;
    $min-pieces = min( $min-pieces, $_.elems ) for @path-pieces;


    my $index = 0;
    while ( $index < $min-pieces ) {
        my $current = @path-pieces[ 0 ][ $index ];
        my $found = True;

        for @path-pieces -> $p {
            $found = False if $current !~~ $p[ $index ];
        }
        @common-path.push: $current if $found;
        $index++;
    }

    @common-path.unshift: '/';
    @common-path.join( '/' ).subst( '//', '/' ).say;
}


```
<br/>
<br/>

The `@common-path` contains all the containing directory that every path has in common with all the others. The loop extracts the current directory from the first path and checks it against the same position in all the other paths. It would suffice to exit the `while` loop as soon as a non match is found, because it does not make any sens to go any further, but this is a small improvement I've not placed into the code.

<a name="task1plperl"></a>
## PWC 182 - Task 1 in PostgreSQL PL/Perl

This time I used a loop to keep track of the greatest value and its position within the list:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc182.task1_plperl( int[] )
RETURNS int
AS $CODE$
   my ($index, $max_value);
   my $current_index = -1;
   for ( @{ $_[0] } ) {
       $current_index++;
       ( $index, $max_value ) = ( $current_index, $_ )
                         if ( ! $max_value || $_ > $max_value  );
   }

   return $index;

$CODE$
LANGUAGE plperl;
```
<br/>
<br/>




<a name="task2plperl"></a>
## PWC 182 - Task 2 in PostgreSQL PL/Perl


Similar to the Raku approach:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc182.task2_plperl( text[] )
RETURNS text
AS $CODE$

   my @common_path = ( '/' );
   my $current_piece;
   my $index = 0;
   my $min_index = 99;

   while ( $index < $min_index ) {
       my @current_pieces = split( '/', @{ $_[0] }[ 0 ] );
       $min_index = $#current_pieces if ( $#current_pieces < $min_index );

       my $found = 1;
       for my $current_dir ( @{ $_[0] } ) {
           my @pieces = split( '/', $current_dir );
           $min_index = $#pieces if ( $#pieces < $min_index );
           $found = 0 if ( $pieces[ $index ] != $current_pieces[ $index ] );
       }

       push @common_path, $current_pieces[ $index ] if ( $found && $current_pieces[ $index ] );
       last if ( ! $found );
       $index++;
   }

   return join( '/', @common_path );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

This time I placed the optimization that `last`s the loop at the very first no-match encountered.

<a name="task1plpgsql"></a>
## PWC 182 - Task 1 in PostgreSQL PL/PgSQL

Same approach as in PL/Perl:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc182.task1_plpgsql( n int[] )
RETURNS int
AS $CODE$
DECLARE
        i int;
        index int := -1;
        current_max int := 0;
        current_index int := -1;
BEGIN
        FOREACH i IN ARRAY n LOOP
                index := index + 1;

                IF i > current_max THEN
                   current_max   := i;
                   current_index := index;
                END IF;
        END LOOP;

        RETURN current_index;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


<a name="task2plpgsql"></a>
## PWC 182 - Task 2 in PostgreSQL PL/PgSQL

Similar to the PL/Perl implementation, this time using a single string instead of an array:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc182.task2_plpgsql( paths text[] )
RETURNS text
AS $CODE$
DECLARE
        p text;
        i int;
        pieces text[];
        ok boolean;
        found_pieces text;
        current_pieces text[];
BEGIN
        found_pieces := '';
        i := 1;
        pieces := regexp_split_to_array( paths[1], '/' );


        FOR i IN 1 .. array_length( pieces, 1 )  LOOP
          FOREACH p IN ARRAY paths LOOP
                  current_pieces := regexp_split_to_array( paths[i], '/' );
                  ok := true;
                  IF current_pieces[i] <> pieces[i] THEN
                     ok := false;
                  END IF;
          END LOOP;

          IF ok THEN
             found_pieces := found_pieces || '/' || pieces[i];
          END IF;

        END LOOP;
        RETURN found_pieces;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
