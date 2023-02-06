---
layout: post
title:  "Perl Weekly Challenge 203: Nested Loops"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 203: Nested Loops

This post presents my solutions to the [Perl Weekly Challenge 203](https://perlweeklychallenge.org/blog/perl-weekly-challenge-203/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 203 - Task 1 - Raku](#task1)
- [PWC 203 - Task 2 - Raku](#task2)
- [PWC 203 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 203 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 203 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 203 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 203 - Task 1 - Raku Implementation

The first task was about finding out quadruplets in an array of integers, where the position of the intergers within the array was from left to right, and the sum of the first three values gave the fourth one. This is easy to implement by means of nested loops:

<br/>
<br/>
```raku
sub MAIN( Bool :$verbose = True,  *@list where { @list.grep( * ~~ Int ).elems == @list.elems } )  {
    my @quadruplets;

    for 0 ..^ @list.elems -> $a {
	for $a ^..^ @list.elems -> $b {
	    for $b ^..^ @list.elems -> $c {
			for $c ^..^ @list.elems -> $d {
			    my ( $la, $lb, $lc, $ld ) = @list[ $a ], @list[ $b ], @list[ $c ], @list[ $d ];
			    @quadruplets.push: [ $la, $lb, $lc, $ld ] if ( ( $la + $lb + $lc ) == $ld );
			}
	    }
	}
    }

    @quadruplets.join( "\n -> " ).say if $verbose;
    @quadruplets.elems.say;
}

```
<br/>
<br/>

Thanks to nested loops, the `$la`..`$ld` values are choosen from left to right, and therefore they are already in the right order.


<a name="task2"></a>
## PWC 203 - Task 2 - Raku Implementation

A strange task: copy only directories from one source folder to another. It is not clear if there is the need to copy only directories and not special files, like links or pipes, and at which recursive level (if any) there is the need to stop.
However, I implemented it using the `IO::Path` role methods:

<br/>
<br/>
```raku
sub MAIN( $src, $dst ) {
    exit if ! $src.IO.d;
    exit if ! $dst.IO.d;

    for $src.IO.dir( test => { ( $src.IO.absolute ~ "/$_" ).IO.d } ) -> $dir {
		# skip . and .. directories
		next if $dir ~~ / ^ \. ** 1..2  $ /;
		$dst.IO.mkdir( $dir.basename );
    }
}

```
<br/>
<br/>

The idea is to iterate over the `$src` directory, using the `test` filter to exclude anything is not a directory. Then I skip everything that starts and ends with one or two dots, to avoid the `.` and `..` folders.
Last, thanks to the `mkdir` methods, I do create the new directory into the `$dst` folder.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 203 - Task 1 - PL/Perl Implementation

This is a mere re-implementation of the Raku version.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc203.task1_plperl( int[] )
RETURNS int
AS $CODE$
  my ( $list ) = @_;
  my @quadruplets;

  for my $a ( 0 .. scalar( $list->@* ) - 1  ) {
    for my $b ( $a + 1 .. scalar( $list->@* ) - 1 ) {
      for my $c ( $b + 1 .. scalar( $list->@* ) - 1 ) {
        for my $d ( $c + 1 .. scalar(  $list->@* ) - 1 ) {
	  my ( $la, $lb, $lc, $ld ) = ( $list->[ $a ], $list->[ $b ], $list->[ $c ], $list->[ $d ] );
	  push @quadruplets, [ $la, $lb, $lc, $ld ] if ( ( $la + $lb + $lc ) == $ld );
        }
      }
    }

  }

  return scalar @quadruplets;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 203 - Task 2 - PL/Perl Implementation

This is a much more complex task to implement in PostgreSQL, since, by design (and for very security reasons), interacting with the local filesystem is hard from within the server.
I decided to use `File::Find` to search for the directories, and `File::Path` to build the paths. **It is important to note that only directotories where the server has access will be scanned/manipulated!**


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc203.task2_plperl( text, text )
RETURNS VOID
AS $CODE$
   use File::Find;
   use File::Path qw/make_path/;

   my ( $src, $dst ) = @_;
   my @paths;

   my $directory_scanner = sub {
      return if ! -d $_;
      return if $_ =~ /^\,{1,2}$/;
      push @paths, "$dst/$_";
   };


   find( { wanted => $directory_scanner } , $src );

   for ( @paths ) {
       make_path( $_ );
   }

   return;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

The idea is to use `$directory_scanner` as a function to populate the `@paths` array with the filenames of the directories to create.
Then, the `make_path` does the job. Please note that I could have passed the whole array to `make_path`, but I decided to iterate because I could want to gather and display some debug information for the specific directory.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 203 - Task 1 - PL/PgSQL Implementation

Same nested loop implementation as the PL/Perl and Raku implementations.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc203.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$
DECLARE
	a int;
	b int;
	c int;
	d int;
	total int := 0;
BEGIN
	FOR a IN  1 .. array_length( l, 1 )  LOOP
	    FOR b IN  a + 1 .. array_length( l, 1 )  LOOP
	    	FOR c IN  b + 1 .. array_length( l, 1 )  LOOP
		    FOR d IN  c + 1 .. array_length( l, 1 )  LOOP
		    	IF l[a] + l[b] + l[c] = l[d] THEN
			   total := total + 1;
			END IF;
		    END LOOP;
		END LOOP;
	    END LOOP;
	END LOOP;

	RETURN total;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 203 - Task 2 - PL/PgSQL Implementation

While it is possible to get a directory content by means of the `pg_ls_dir` function, there is no native way to create a new directory, therefore I decided to call the PL/Perl counterpart to do the job:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc203.task2_plpgsql( src text, dst text )
RETURNS VOID
AS $CODE$
BEGIN
	SELECT pwc203.task2_plperl( src, dst );
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
