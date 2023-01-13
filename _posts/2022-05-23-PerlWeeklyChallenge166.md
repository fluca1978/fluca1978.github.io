---
layout: post
title:  "Perl Weekly Challenge 166: file, directories and hex!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 166: files, directories and hex!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 166](https://perlweeklychallenge.org/blog/perl-weekly-challenge-166/){:target="_blank"}.

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
## PWC 166 - Task 1

Given a dictionary, a text file with a word per-line, find out all the words that can be expressed as an hexadecimal string substituting some letters with numbers.
Each word must be between two and eight chars.
This is my Raku implementation:



<br/>
<br/>
```raku
sub MAIN( Str $filename = '../../data/dictionary.txt', Int $limit = 100 ) {
    my %substitutions = 'o' => 0, 'l' => 1, 'i' => 1, 's' => 5, 't' => 7;

    my @results = lazy gather {
        for $filename.IO.lines -> $word is copy {
            next if $word.chars > 8 || $word.chars < 2;
            next if $word.lc !~~ / ^ <[a..folist]>+ $ /;

            $word .= lc;
            my $src-word = $word;

            #$word .= subst( $_, %substitutions{ $_ } ) for %substitutions.keys;
            for %substitutions.kv -> $k, $v {
                $word ~~ s:g/$k/$v/;
            }

            take [ $src-word.uc, $word.uc ];
        }
    };


    @results[0 .. $limit].map( { "The word $_[0] is translated to $_[1]" } ).join( "\n" ).say;
}


 ```
<br/>
<br/>

I use a `lazy gather` to provide every new word. First, I skip any `$word` that has not hexadceimal letters (i.e, not in the range 'a'..'f') and that do not contain substituble letters.
<br/>
Then I do an iteration over all possible substitutions and do a regexp global substitution (that is the same as the `subst` commented out line). Last, I return an array with the original word and the substituted one.
<br/>
The last line does only a pretty printing of the result.



<a name="task2"></a>
## PWC 166 - Task 2

Given three directories, find out which file are not simultaneously available in all of them.

<br/>
<br/>
```raku
sub MAIN( Str $dir-a, Str $dir-b, Str $dir-c ) {


    my @dirs = $dir-a, $dir-b, $dir-c;
    my %files;

    # build the directory content
    for @dirs -> $current-dir {
        %files{ $_.basename  ~ ( $_.d ?? '/' !! '' ) }.push: $current-dir for $current-dir.IO.dir( test => { $current-dir.IO.d || $current-dir.IO.f } );
    }


    # print the stuff
    my $header = False;
    for %files.kv -> $file-name, $dir-names {
        # skip all entries that are there for all the directories!
        next if %files{ $file-name }.elems == @dirs.elems;

        "|%-20s|%-20s|%-20s|\n|%s|%s|%s|".sprintf( @dirs, ( '-' x 20 ) xx @dirs.elems ).say and $header = True if ! $header;
        printf "|%-20s", $dir-names.grep( $_ ) ?? $file-name !! ' '         for @dirs;
        print "|";

        say "";
    }

}


```
<br/>
<br/>

First of all, I scan every directory and place the files as keys in an `%files` hash. Each entry in the hash contains an array with the directories where the file is present. Therefore, given every file, the entries indicates in which directories the file is present.
<br/>
Next I iterate over all the found files, and the number of directories where the file is found is the same as the number of directories, then I can skip its printing (i.e., the file is everywhere). On the other hand, I print a formatted string with the directory name or an empty string if the file is not within a directory.

<a name="task1plperl"></a>
## PWC 166 - Task 1 in PostgreSQL PL/Perl

Same implementation as Raku, but using `plperlu` since there is the need to access the filesystem.


<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc166.task1_plperl( text, int, int )
RETURNS SETOF text
AS $CODE$
my ($filename, $min, $max) = @_;
my $line;
my $substitutions = { o => 0,
                      l => 1,
                      i => 1,
                      s => 5,
                      t => 7 };

open my $file, "<", $filename;

while ( $line = <$file> ) {
      chomp $line;
      next if length( $line ) < $min || length( $line ) > $max;

      $line = lc $line;
      next if $line !~ /^[a..folist]+$/;

      for my $k ( keys( %$substitutions ) ) {
          $line =~ s/$k/$substitutions->{ $k }/g;
      }

      return_next( $line );
}

close $file;
return undef;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

*I miss auto-chomping*! In fact, in the first implementation I was unable to get a word-per-line.


<a name="task2plperl"></a>
## PWC 166 - Task 2 in PostgreSQL PL/Perl

Similar to the Raku implementation:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc166.task2_plperl( text, text, text )
RETURNS TABLE( dir_a text , dir_b text , dir_c text )
AS $CODE$
my ( $dir_a, $dir_b, $dir_c ) = @_;
my $files = {};
my @dirs;
my $index = -1;

for my $current_dir ( @_ ) {
  elog( INFO, "Opening $current_dir" );
  $index++;
  push @dirs, "dir_" . ( 'a' .. 'c' )[ $index ];
  opendir my $dir, $current_dir;
  while ( my $entry = readdir( $dir ) ) {
        my $filename = $entry . ( -f "$current_dir/$entry" ? "" : "/" );
        push $files->{ $filename }->@*, "dir_" . ( 'a' .. 'c' )[ $index ];
  }
  closedir $dir;
}

for my $filename ( sort keys( $files->%* ) ) {
    next if scalar( $files->{ $filename }->@* ) == scalar @_;
    my $hash = {};

    $hash->{ $_ } = undef for @dirs;
    for my $dir ( $files->{ $filename }->@* ) {
        $hash->{ $dir } = $filename;
    }

return_next( $hash );
}


return undef;

$CODE$
LANGUAGE plperlu;
```
<br/>
<br/>

In this implementation I use `opendir` and `readdir` functions to get the listing (i.e., content) as filenames. One difference between the Raku implementation is that in Perl 5 `readdir` returns a relative filename.
<br/>
Once the hash with the files is built, I iterate over all the files skipping those that are present ion the same number of occurrencies as the number of the directories, and then prefill an `$hash` with the values.
<br/>
Since we need to keep the list of columns constant, I remap the directories into the `$files` hash with a trick like `"dir_" . ( 'a' .. 'c' )[ $index ]` that maps any directory name into `dir_a` thru `dir_c`.

<a name="task1plpgsql"></a>
## PWC 166 - Task 1 in PostgreSQL PL/PgSQL

Just a nested query, no need to use PL/PgSQL:

<br/>
<br/>

``` sql
create table if not exists pwc166.dictionary( word text );
truncate pwc166.dictionary;
copy pwc166.dictionary from '/tmp/dictionary.txt';

SELECT word,
        regexp_replace(
          regexp_replace(
             regexp_replace(
                regexp_replace( lower( word ), 'o', '0', 'g' ),
                'l|i', '1', 'g' ),
                  's', '5', 'g' ),
                  't', '7', 'g' )
FROM pwc166.dictionary
WHERE
length( word ) >= 2
AND length( word ) <= 8
AND word ~* '^[a-folist]+$'
;
```
<br/>
<br/>

The idea is to populate the `dictionary` table with the content of the `dictionary.txt` file (via `COPY`). Then the query selects all words within two and eight chars that match a case-insensitive regular expression with the hex letters (via `~*`).
<br/>
Then, I nest `regexp_replace()` calls to change every single letter globally.

<a name="task2plpgsql"></a>
## PWC 166 - Task 2 in PostgreSQL PL/PgSQL

Another "simple" query:

<br/>
<br/>

``` sql
WITH
dir_a(f) AS
(
   SELECT *
   FROM pg_ls_dir( '/tmp/dir_a', TRUE, false)
)
, dir_b(f) AS
(
SELECT *
FROM pg_ls_dir( '/tmp/dir_b', true, false)
)
, dir_c(f) AS
(
SELECT *
FROM pg_ls_dir( '/tmp/dir_c', true,false)
)
SELECT a.f as dir_a, b.f as dir_b, c.f as dir_c
FROM dir_a a
FULL OUTER JOIN dir_b b on a.f = b.f
FULL OUTER JOIN dir_c c on c.f = a.f
WHERE
a.f IS NULL
OR b.f IS NULL
OR c.f IS NULL
;

```
<br/>
<br/>

The three CTEs provide a list of files within each directory using `pg_ls_dir()`. Then I do a full outer join on all the materialzed tables where any of the elements is `NULL`.
<br/>
This implementation has a little drawback: it cannot discriminate between a file and a directory with the same name.
