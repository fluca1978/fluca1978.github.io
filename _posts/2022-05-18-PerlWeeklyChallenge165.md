---
layout: post
title:  "Perl Weekly Challenge 165: SVG"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 165: SVG

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 165](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0165/){:target="_blank"}.

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
## PWC 165 - Task 1

This task was about to plot a set of points and lines inserted from the standard input.
I used the `SVG` module, that in turns is a wrapper to produce an XML document that represents an SVG image.



<br/>
<br/>
```raku
use SVG;
sub MAIN( Str $filename = 'task1.svg' ) {
    my ( @points, @lines );

    for $*IN.lines() -> $line {
        my @elements = $line.split(',').map( *.trim );
        next if @elements.elems !%% 2 && @elements.elems !%% 4;

        if @elements.elems == 2 {
            # a point
            my $point = circle =>  [ cx => @elements[ 0 ].Num,
                                     cy => @elements[ 1 ].Num,
                                     r => 5,
                                     fill => 'blue' ];
            @points.push: $point;
        }
        else {
            # a line
            my $line = line => [ x1 => @elements[ 0 ].Num,
                                 y1 => @elements[ 1 ].Num,
                                 x2 => @elements[ 3 ].Num,
                                 y2 => @elements[ 3 ].Num,
                                 stroke => 'magenta' ];
            @lines.push: $line;
        }

    }

    $filename.IO.spurt( SVG.serialize:
            svg => [ width => 500, height => 500, |@points, |@lines ] );
}

 ```
<br/>
<br/>

The idea is quite simple. First of all I split the input into number pairs and if the user has inserted something that is not a number nor a correct pair, I skip the input line.
Then I check: if there re only two numbers, it is a point, otherwise it must be a line.
Depending on that, I create an hash containing the coordinates with names as they have to appear in the SVG graphic (here I took some examples before I was ready to plo!).
Last I ask `SVG` to *plot* the result in a `500x500` canvas. The resulting XML is `spurt`ed to a file, so that the image is effectively stored on the hard drive.


<a name="task2"></a>
## PWC 165 - Task 2

A task to plot the nearest line for a set of points using the previous program.

<br/>
<br/>
```raku
sub MAIN() {

    my @input-points =
                <333,129  39,189 140,156 292,134 393,52  160,166 362,122  13,193
                 341,104 320,113 109,177 203,152 343,100 225,110  23,186 282,102
                 284,98  205,133 297,114 292,126 339,112 327,79  253,136  61,169
                 128,176 346,72  316,103 124,162  65,181 159,137 212,116 337,86
                 215,136 153,137 390,104 100,180  76,188  77,181  69,195  92,186
                 275,96  250,147  34,174 213,134 186,129 189,154 361,82  363,89 >;

    # "decompose" the input into two elements arrays
    my @points = @input-points.split( / \s+ / ).split( ',' ).split( / \s+ / ).map( *.Int ).rotor: 2;


    # compute all the parts
    my ( $m, $x, $y, $xy, $xx ) = 0,0,0,0,0;
    $x  = [+] @points.map( { $_[0] } );
    $y  = [+] @points.map( { $_[1] } );
    $xy = [+] @points.map( { $_[0] * $_[1] } );
    $xx = [+] @points.map( { $_[0] ** 2 } );
    $m  = ( @points.elems * $xy - $x * $y ) / ( @points.elems * $xx - $x ** 2 );

    my $b = 0;
    $b = ( $y - $m * $x ) / @points.elems;

    # compute the line start and end point
    my @line;
    @line.push: $_, $m * $_ + $b for 0,100;


    # now I need to graph
    my $task1 = run "raku", <raku/ch-1.p6 task2.svg>, :in, :err;
    $task1.in.say: $_.join( ',' ) for @points;
    $task1.in.say: @line.join( ',' );
    $task1.in.close;
    $task1.err.slurp.say;
}

```
<br/>
<br/>

First of all, `@points` is an array of couple of points, so that the previous program can use it as its input. For gaining such couples, there is a little mangling to be done, with particular regard to spaces.
<br/>
Then I compute the `y = mx + b` formula, or better `$m` and `$b` and I compute the starting and ending points using `x` with `0` and `100`.
With all that set, I can now invoke the previous program using Raku process faciliies and passing all the points and then the ling as `:in` standard input.


<a name="task1plperl"></a>
## PWC 165 - Task 1 in PostgreSQL PL/Perl

A quite straightforward implementation: first I put the input points into the `points` table, then let's iterate on such table:


<br/>
<br/>

``` sql
CREATE TABLE IF NOT EXISTS points( x1 int, y1 int, x2 int, y2 int );

TRUNCATE points;

INSERT INTO points
VALUES
(53,10, NULL, NULL)
,(53,10,23,30)
,(23,30, NULL, NULL)
;

--
-- This function generates the SVG XML document
-- from the 'points' table.
--
CREATE OR REPLACE FUNCTION
pwc165.plperl_generate_svg_xml( text )
RETURNS TEXT[]
AS $CODE$

   my ( $filename ) = @_;

   my @lines;
   my @points;

   my $result_set = spi_exec_query( 'SELECT * FROM points;' );
   for my $row_number ( 0 .. $result_set->{ processed } - 1 ) {
       my $row = $result_set->{ rows }[ $row_number ];

       # if it has a single couple, it is a point
       my ( $x1, $y1, $x2, $y2 ) = map { $row->{ $_ } } qw<x1 y1 x2 y2>;
       my $is_line = $x1 && $y1 && $x2 && $y2;

       push @points, [ $x1, $y1 ] if ! $is_line && $x1 && $y1;
       push @points, [ $x2, $y2 ] if ! $is_line && $x2 && $y2;
       push @lines, [ $x1, $y1, $x2, $y2 ] if $is_line;

   }


   my $svg = q{<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
   <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
   <svg height="400" width="400" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg"> };

   for my $line ( @lines ) {
      $svg .=  sprintf( '<polyline points="%s" stroke="#ff0000" stroke-width="6" />', join( ' ', @$line ) );
   }

   for my $point ( @points ) {
     $svg .= sprintf( '<circle r="4" cx="%d" cy="%d" stroke-width="0" fill="#000000" />', $point->[ 0 ], $point->[ 1 ] );
   }


   if ( $filename ) {
      open my $fh, ">", $filename || die "Cannot open $filename !";
      print { $fh } $svg;
      close $fh;
   }

   return( [ $svg, $filename ] );

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

as you can see, I do query `points` and decide if it is a line when there are all four values, otherwise if only a couple is present (either `x1,y1` or `x2,y2`) I consider it a point.
<br/>
Then I compose the SVG XML storing it into the `$svg` variable.
<br/>
Last, if a filename `$filename` has been specified, I spurt the content into such file. **For writing on the filesystem, I need to use `plperlu`!++


<a name="task2plperl"></a>
## PWC 165 - Task 2 in PostgreSQL PL/Perl

The second task is built on top of the first one, so the `points` table is truncated and inserted with the only `x1, y1` values (points), then all the mathematics is computed in the same manner as the Raku implementation.
Last, a line with four points is inserted into the `points` table and the previous function is called to generate the SVG file:


<br/>
<br/>

``` sql
TRUNCATE points;

INSERT INTO points( x1, y1 )
VALUES
(333,129) ,( 39,189) ,(140,156) ,(292,134) ,(393,52 ) ,(160,166) ,(362,122) ,( 13,193)
,(341,104) ,(320,113) ,(109,177) ,(203,152) ,(343,100) ,(225,110) ,( 23,186) ,(282,102)
,(284,98)  ,(205,133) ,(297,114) ,(292,126) ,(339,112) ,(327,79 ) ,(253,136) ,( 61,169)
,(128,176) ,(346,72 ) ,(316,103) ,(124,162) ,( 65,181) ,(159,137) ,(212,116) ,(337,86 )
,(215,136) ,(153,137) ,(390,104) ,(100,180) ,( 76,188) ,( 77,181) ,( 69,195) ,( 92,186)
,(275,96)  ,(250,147) ,( 34,174) ,(213,134) ,(186,129) ,(189,154) ,(361,82 ) ,(363,89 )
;


CREATE OR REPLACE FUNCTION
pwc165.task2_plperl( text )
RETURNS TEXT
AS $CODE$

my ( $filename ) = @_;
my ( $m, $x, $y, $xy, $xx ) = 0,0,0,0,0;
my @points;

my $result_set = spi_exec_query( 'select x1, y1 from points' );
for my $index ( 0 .. $result_set->{ processed } - 1 ) {
  my $row = $result_set->{ rows }[ $index ];
  push @points, [ $row->{ x1 }, $row->{ y1 } ];
}


$x  += $_->[ 0 ] for ( @points );
$y  += $_->[ 1 ] for ( @points );
$xy += $_->[ 0 ] * $_->[ 1 ] for ( @points );
$xx += $_->[ 0 ] * $_->[ 0 ] for ( @points );
$m   = ( $#points * $xy - $x * $y ) / ( $#points * $xx - $x * $x );

my $b = 0;
$b = ( $y - $m * $x ) / $#points;

elog( DEBUG, "y = $m * x + $b" );

# compute two points in the line
my ( $x1, $y1, $x2, $y2 ) = ( 0, $b, 100, 100 * $m + $b );
# insert the line
spi_exec_query( "INSERT INTO points( x1, y1, x2, y2 ) VALUES( $x1, $y1, $x2, $y2 ); " );

# now call the other function to plot the graph
spi_exec_query( sprintf "SELECT pwc165.plperl_generate_svg_xml( '%s' );", $filename );
return( $filename );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


please note the usage of `spi_exec_query` to both query and update the database status.


<a name="task1plpgsql"></a>
## PWC 165 - Task 1 in PostgreSQL PL/PgSQL

Cloned PL/Perl implementation:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc165.task1_plpgsql()
RETURNS TEXT
AS $CODE$
DECLARE
        svg text;
        p points%rowtype;
BEGIN
        SELECT '<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
<svg height="400" width="400" xmlns="http://www.w3.org/2000/svg" xmlns:svg="http://www.w3.org/2000/svg">'
       INTO svg;

       FOR p IN SELECT * FROM points LOOP
               IF p.x1 IS NOT NULL AND p.y1 IS NOT NULL AND p.x2 IS NOT NULL AND p.y2 IS NOT NULL THEN
                  -- line
                  SELECT svg
                         || format( '<polyline points="%s %s %s %s" stroke="#ff0000" stroke-width="6" />',
                                     p.x1, p.y1, p.x2, p.y2 )
                 INTO SVG;
              END IF;

              IF p.x1 IS NOT NULL AND p.y1 IS NOT NULL THEN
                 -- point
                 SELECT svg
                        || format( '<circle r="4" cx="%s" cy="%s" stroke-width="0" fill="#000000" />', p.x1, p.y1 )
                 INTO svg;
              END IF;

              IF p.x2 IS NOT NULL AND p.y2 IS NOT NULL THEN
                 -- point
                 SELECT svg
                        || format( '<circle r="4" cx="%s" cy="%s" stroke-width="0" fill="#000000" />', p.x2, p.y2 )
                INTO svg;
             END IF;
       END LOOP;

RETURN svg;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I do use `format` to construct a well formatted string in the XML SVG content.
Please note that there is no way to write to a file from PL/PgSQL, so what you can do is redirect the query to a file output or write another function (e.g., PL/Perl) to write to a file for you.


<a name="task2plpgsql"></a>
## PWC 165 - Task 2 in PostgreSQL PL/PgSQL

Again, a reimplementation of the PL/Perl solution:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc165.task2_plpgsql()
RETURNS TEXT
AS $CODE$
DECLARE
        x  float := 0;
        y  float := 0;
        xx float := 0;
        xy float := 0;
        m  float := 0;
        b  float := 0;
        c  int   := 0;

        p  points%rowtype;
BEGIN

        FOR p IN SELECT x1, y1 FROM points LOOP
            x  := x + p.x1;
            y  := y + p.y1;
            xy := xy + p.x1 + p.y1;
            xx := xx + p.x1 * p.x1;
            c  := c + 1;
        END LOOP;

        m := ( c * xy - x * y ) / ( c * xx - x * x );
        b := ( y - m * x ) / c;

        INSERT INTO points( x1, y1, x2, y2 )
        SELECT 0, b, 100, 100 * m + b;

        RETURN pwc165.task1_plpgsql();
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

See how simple it is to return the value of the previous task function in this case: PL/PgSQL does not require you to query the database against a function, rather to just invoke such function!
