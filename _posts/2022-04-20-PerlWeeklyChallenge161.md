---
layout: post
title:  "Perl Weekly Challenge 161: words"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 161: words

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 161](https://perlweeklychallenge.org/blog/perl-weekly-challenge-161/){:target="_blank"}.

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
## PWC 161 - Task 1

The first task was about finding out *abecedarian* words from an established dictionary. The idea is to find out if a word has a sequence of letter already sorted. This is not that much complex in Raku, and in fact, part of the code is to check out if the dictionary file has been given correctly or not.


I check every word in the dictionary splitting it into its set of letters, than sorting such letters and re-joining them into a word: if the initial word is equal to the obtained one than the letters are sorted.
<br/>
I place the word into an array within an hash, the hash is keyed by the length of the word. This will allow me to print all the words with the same length into a block.



<br/>
<br/>
```raku

sub MAIN( Str $dictionary-file-name? = '../../data/dictionary.txt' ) {
    die "\nCannot find the dictionary [$dictionary-file-name]\n" if ! $dictionary-file-name || ! $dictionary-file-name.IO.f;

    # store the length and the list (array) of words for such length
    my %abecedarian-words;

    for $dictionary-file-name.IO.lines -> $word {
        # a word is abecedarian if the sorted letter composed word
        # is equal to the word itself
        %abecedarian-words{ $word.chars }.push: $word if ( $word.comb.sort.join ~~ $word );
    }

    # print the length and the list of words
    "( $_ ):\n { %abecedarian-words{ $_ }.join( ',' ) }".say for %abecedarian-words.keys.sort;

}

 ```
<br/>
<br/>


<a name="task2"></a>
## PWC 161 - Task 2

This task was about producing *pangrams*, a sentence where the words make the appearance of at least every letter in the alphabet. This a simplistic implementation, since it is not required to produce a sentence with a sense!
<br/>
The task needs to use the same dictionary provided by the challenge.

<br/>
<br/>
```raku
sub MAIN( Str $dictionary? = '../../data/dictionary.txt' ){
    die "\nCannot find dictionary [$dictionary]" if ( ! $dictionary || ! $dictionary.IO.f );

    # read all the words in alphabetical order
    my @words = $dictionary.IO.lines.sort;


    my @letters =  'a' .. 'z' ;
    my @found;

    for @letters {
        # avoid to search a word if I've already one that
        # covers the current letter
        next if @found.grep( * ~~ / $_ / );

        # now search a new word
        @found.push: @words.grep( * ~~ / $_ / ).pick;
    }

    # all done
    @found.join( ' ' ).say;
}

```
<br/>
<br/>

Here I do iterate over all the letters, and search for a word that contains the current letter `$letter` only if I've not found already a word that contains such letter.
I push every word into an array `@found` and then print it as a sentence.


<a name="task1plperl"></a>
## PWC 161 - Task 1 in PostgreSQL PL/Perl


*Please note that I've used a `dictionary` table that contains, one tuple per line, the words from the input file. I'm gonig to use such table in all the PostgreSQL related examples!*
In order to load the table easily, you can do something like the following:

<br/>
<br/>

``` shell
% psql -h miguel -U luca -c 'create schema pwc161;' testdb
% psql -h miguel -U luca -c 'create table pwc161.dictionary( word text );' testdb
$ psql -h miguel -U luca \
       -c 'COPY pwc161.dictionary( word ) FROM STDIN;' testdb \
              < ../../data/dictionary.txt
```
<br/>
<br/>

The you are going to have a table with a single column, `word` that contains all the same data of the already mentioned text file.

<br/>
<br/>

The implementation of this task is pretty much the same as the Raku counterpart:

<br/>
<br/>
``` sql
CREATE OR REPLACE FUNCTION
pwc161.plperl_abecedarian()
RETURNS SETOF text
AS $CODE$

 my $query = spi_query( 'SELECT word FROM pwc161.dictionary ORDER BY length( word ) ASC' );
 while ( defined ( $row = spi_fetchrow( $query ) ) ) {
  my @letters = split //, $row->{ word };
    return_next( $row->{ word } ) if ( $row->{ word } eq join( '', sort( @letters ) ) );
}

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


The words are extracted via a specific query using the `spi_query` function, that prepares the query, and then a tuple (i.e., a word) at a time is extracted via `spi_fetchrow` internal function.
The word is contained into `$row->{ word ]` and I compare it against the `join` of the `split` into letters that are `sort`ed. If they match, the function returns a new value appending it into the result set.




<a name="task2plperl"></a>
## PWC 161 - Task 2 in PostgreSQL Pl/Perl

An implementation similar to the Raku one:

<br/>
<br/>

``` perl
CREATE OR REPLACE FUNCTION pwc161.pangrams()
RETURNS SETOF text
AS $CODE$
 my @found;

 for my $letter ( 'a' .. 'z' ){
   my $query = spi_query( "SELECT word FROM pwc161.dictionary WHERE word like '%$letter%' order by random()" );

 while ( defined ( $row = spi_fetchrow( $query ) ) ) {
  # first word ever
  push @found, $row->{ word } and return_next( $row->{ word } ) if ! @found;

  next if grep /$letter/, @found;

  # add the word
  push @found, $row->{ word };
  return_next( $row->{ word } );

  }
}

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I do loop on every letter, and for each letter I do a single query to get a randomly chosen word that contains such letter. If I've already found such letter, than I skip to the next letter, otherwise I append the word to the result set. The aim of the `@found` array is just to smake the decision about seen words and letters, and thus to skip to another letter.


<a name="task1plpgsql"></a>
## PWC 161 - Task 1 in PostgreSQL Pl/PgSQL

The task can be solved with a single SQL query:

<br/>
<br/>

``` sql
WITH words( word, size, sorted, unsorted )
AS (
       select word, length( word ),
        array( select regexp_split_to_table( word, '' ) order by 1 ) as sorted,
        regexp_split_to_array( word, '' ) as unsorted
       from pwc161.dictionary )
select word, size
from words
where sorted = unsorted
order by word, size asc;
```
<br/>
<br/>

The `words` CTE provides every word, along with its size (in chars), its list of letters (as unsorted array) and its listed list of letters. Note how the trick to sort the letters is to split the rod into a table, sort by `ORDER BY` and reconstruct it as an array with the `ARRAY` constructor.
<br/>
Having doing this, it is possible to extract the words where the sorted and unsorted arrays have the same value (the `=` operator against arrays provide a deep element-by-element comparison), sorting it by the length of each word.

<a name="task2plpgsql"></a>
## PWC 161 - Task 2 in PostgreSQL Pl/PgSQL

This is a straightforward implementation that reminds the PL/Perl one:

<br/>
<br/>

``` sql
CREATE OR REPLACE FUNCTION
pwc161.pangrams_plpgsql()
RETURNS SETOF TEXT
AS $CODE$
DECLARE
        start_letter int;
        end_letter   int;
        letter char;
        current_word pwc161.dictionary%rowtype;
        found_words text;
BEGIN
  start_letter := ascii( 'a' );
  end_letter   := ascii( 'z' );
  FOR l IN start_letter .. end_letter LOOP
      letter := chr( l );

      IF found_words LIKE '%' || letter || '%' THEN
         CONTINUE;
      END IF;

      SELECT word
      INTO   current_word
      FROM pwc161.dictionary
      WHERE word like '%' || letter || '%'
      ORDER BY RANDOM()
      LIMIT 1;



      found_words := found_words || ' ' || current_word;
      RETURN NEXT current_word.word;

  END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>

Since in PostgreSQL PL/PgSQL the `FOR` can loop only over ranges of integers, I use two variables `start_letter` and `end_letter`, to contain the ASCII value of the letters 'a' and 'z'. Then, with the `chr()` function I can get back a character from a codepoint (well, an ASCII value).
<br/>
After that I extract a randomly chosen single word from the `dictionary` table, and put it into the `current_word` variable. It is just a matter to append the current extracted word to the result set to get the job done.
The aim of `found_words` is to represent a huge string with the selected-so-far words in order to be able to skip the searching for a word if a letter has been already coverd.
