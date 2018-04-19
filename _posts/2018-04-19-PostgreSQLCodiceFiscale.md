---
layout: post
title:  "Generating an italian 'Codice Fiscale' via plpgsql or plperl"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- itpug
permalink: /:year/:month/:day/:title.html
---
PostgreSQL built-in `plpgsql` can be used to build *stored procedure* and, with a few tricks, to consume data and translate it into other forms. It is also possible to generate a so known *codice fiscale*, the italian string that represents the *tax payer number* based on the person's name, birth date and place.
This posts will show some concepts about how to generate the single pieces of the *codice fiscale* via `plpgsql`.
And why not? Let's compare it to a `plperl` implementation.


# Generating an italian *codice fiscale*

In order to provide a quite complet example of usage of `plpgsql` for a course of mine, I developed a few functions to build up an italian *codice fiscale* (tax payer number). The idea is not to have a fully working implementation, rather to demonstrate usage of different operators and functions in `plpgsql`. And to compare its implementation with a `plperl` one.

The full rules for building up a *codice fiscale* are available [here in italian](https://it.wikipedia.org/wiki/Codice_fiscale). The code shown below is freely available on my [GitHub PostgreSQL-related repository](https://github.com/fluca1978/fluca1978-pg-utils), and in particular there are two scripts:
- a [pure `plgpsql` script](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/codice_fiscale.sql);
- a [`plperl` implementation](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/codice_fiscale.plperl.sql).

In order to generate a full "codice fiscale" you need to extract some letters from the surname and the name, build a string representing both the date of birth and gender, a code representing the birth place and last comes a character that works as a *checksum* of all the previous parts.

In order to obtain the full result the following function can be used:

```sql
CREATE OR REPLACE FUNCTION cf.cf( surname text,
                         name text,
                         birth_date date,
                         birth_place text,
                         gender bool DEFAULT true )
RETURNS char(16)
AS $CODE$
DECLARE
  cf_string char(15);
  cf_check  char(1);
BEGIN
  cf_string := cf.cf_letters( surname )
               || cf.cf_letters( name, true )
               || cf.cf_date( birth_date, gender )
               || cf.cf_place( birth_place );

   cf_check := cf.cf_check( cf_string );
   RAISE DEBUG 'cf: % + %', cf_string, cf_check;

  RETURN upper( cf_string || cf_check );
END
$CODE$
LANGUAGE plpgsql;
```

As simple as such function is, it does a *simple* string concatenation invoking other functions.

## Extracting letters for name and surname

The `cf_letters()` function performs the extraction of the letters for either a name or surname. General rules are:
- the result will be three letters wide;
- if possible, only consonants will be used taking them from left to right. If there are not enough consonants, vowels must be appended in the same order they appear;
- in the case of a name, if there are enough consonants, the first, third and fourth must be choosen.

That produces the following code:

```sql
CREATE OR REPLACE FUNCTION cf.cf_letters( subject text, is_name bool DEFAULT false )
RETURNS char(3)
AS $BODY$
DECLARE
  missing_chars int := 0;
  vowels text;
  consonants text;
  final_chars char(3);
BEGIN
  -- work always uppercase, avoid case-sensitiveness
  subject := upper( trim( subject ) );
  -- get all the consonants
  consonants := translate( subject, 'AEIOU', '' );
  -- extract all the vowels (negate consonants!)
  vowels := translate( subject, consonants, '' );

  RAISE DEBUG 'cf_letters: [%] -> [%] + [%]',
                             subject,
                             consonants,
                             vowels;

  IF is_name THEN 
    IF length( consonants ) >= 4 THEN
       consonants := substring( consonants FROM 1 FOR 1 )
                        || substring( consonants FROM 3 FOR 1 )
                        || substring( consonants FROM 4 FOR 1 );
    ELSIF length( consonants ) = 3 THEN
      consonants := substring( consonants FROM 1 FOR 3 );
    END IF;
  END IF;

  IF length( consonants ) >= 3 THEN
     final_chars := substring( consonants FROM 1 FOR 3 );
  ELSE
     missing_chars := 3 - length( consonants );
     RAISE DEBUG 'Pushing % vowel(s)', missing_chars;
     final_chars := consonants || substring( vowels FROM 1 FOR  missing_chars );
  END IF;

  RETURN final_chars;
END
$BODY$
LANGUAGE plpgsql;
```

The function builds two strings: one made up by all the consonants and one made up by only vowels appearing in the string passed as input. After that, in the case it is a name and has enough consonants, the latters are used to build up the three required chars. In any case, if there are not enough consonants, vowels are appended computing first how many letters are *missing* from the final length of `3`.

The same alghoritm results a little shorter in Perl, mainly due to regular expressions and array slicing:

```perl
CREATE OR REPLACE FUNCTION cf.cf_letters( text,  bool DEFAULT false )
RETURNS char(3)
AS $BODY$
  my ($subject, $is_name) = ( lc( $_[ 0 ] ), $_[ 1 ] );

  # split the word into letters
  my @letters = split //, $subject;
  # grep out vowels and consonants
  my @consonants = grep { $_ !~ /[aeiou]/ } @letters;
  my @vowels     = grep { $_ =~ /[aeiou]/ } @letters;

  return join( '', $consonants[0], $consonants[2], $consonants[3] )  if ( $is_name && @consonants >= 4 );
  return join( '', @consonants[0..2] )  if ( @consonants >= 3 );
  return join( '', @consonants, @vowels[0 .. 3 - @consonants - 1] );
$BODY$
LANGUAGE plperl;
```

## Generating the birth date and gender part

The rules are quite simple:
- the year comes first expressed as two digits;
- a capital letter specifies the month;
- the day of birth comes then, expressed as two digits and added by `40` in the case of female gender.

Therefore a birth date like `1978-07-19` becomes `78L19` (where `L` is the letter representing `July`).
In order to compute the date string it is required to know, as input parameters, the date of birth and the gender.
The function that implements the computation is the following one:

```sql
CREATE OR REPLACE FUNCTION cf.cf_date( birth_date date,
                                       male boolean DEFAULT true )
RETURNS char(5)
AS $CODE$
DECLARE
   y int;  -- year
   m int;  -- month (1-12)
   d int;  -- day
   month_decode char[] := ARRAY[ 'a'   -- january
                                 , 'b' -- february
                                 , 'c' -- march
                                 , 'd' -- april
                                 , 'e' -- may
                                 , 'h' -- june
                                 , 'l' -- july
                                 , 'm' -- august
                                 , 'p' -- september
                                 , 'r' -- october
                                 , 's' -- november
                                 , 't' -- december
                               ]::char[];

BEGIN
  -- get the year, last two digits
  y := to_char( birth_date, 'yy' );
  -- get the month index
  m := EXTRACT( month FROM birth_date );
  -- get the day
  d := EXTRACT( day FROM birth_date );

  -- if this is for a female, add
  -- a number to the day
  IF NOT male THEN
    d := d + 40;
  END IF;

  RAISE DEBUG 'cf_date: % -> % % [%] % ',
                          birth_date,
                          y,
                          m,
                          month_decode[m],
                          d;


  -- compose and return the string
  RETURN lpad( y::text, 2, '0' )
            || upper( month_decode[m] )
            || lpad( d::text, 2, '0' );
END
$CODE$
LANGUAGE plpgsql;
```

After extracting the single parts of the `birth_date`, all the parts are composed in a final string using `lpad()` to prepend required `0` if the number is not a two-digit one.

The tricky part of the function is the decoding of the month, from an integer value to a letter. In order to provide a *poor-man map* functionality, I placed all the month letters in their order into an array, so that given a numeric month value (e.g., `7`) I can extract the corresponding letter (`l` for *july*).


## Generating the birth place code

This is a quite boring task, since there is no computation involving. A *simple lookup* is required, so I implemented it with a *simple name lookup* thru a table.

```sql
CREATE TABLE IF NOT EXISTS cf.places( 
   code char(4) PRIMARY KEY,
   description text NOT NULL,
   UNIQUE( description ),
   EXCLUDE( lower( trim( description ) ) WITH = ) );

CREATE OR REPLACE FUNCTION cf.cf_place( birth_place text )
RETURNS char(4)
AS $CODE$
DECLARE
  birth_code char(4);
BEGIN
  SELECT code
  INTO birth_code  -- no strict! allow NOT FOUND to work!
  FROM cf.places
  WHERE lower( description ) = lower( birth_place );

  IF NOT FOUND THEN
     RAISE WARNING '% not in cf.places!', birth_place;
     RETURN 'XXXX';
  END IF;

  RETURN birth_code;
END
$CODE$
LANGUAGE plpgsql;
```

Again, the `plperl` version of the function is not as much shorter as one could expect:

```perl
CREATE OR REPLACE FUNCTION cf.cf_place( text )
RETURNS char(4)
AS $CODE$
   my ($birth_place) = @_;

   my $query = "SELECT code FROM cf.places WHERE lower( '$birth_place' ) = lower( description ) ";
   my $result_set = spi_exec_query( $query , 1);
   return $result_set->{rows}[0]->{ code } if ( $result_set->{ rows } );
   return 'XXXX';

$CODE$
LANGUAGE plperl;

```


## Generating the checksum character

This is even more boring than generating the place code. The rules are that each character is assigned a value depending on  both its position within the string (even or odd) and its actual value. All the values are summed and the result is computed `modulo 26`, and the result is looked up within the help of another table.

So, the whole has been implemented with a lookup table and a fat loop over all the character of the string:

```sql
CREATE TABLE IF NOT EXISTS cf.check_chars( 
   c char PRIMARY KEY,
   odd_value int NOT NULL,
   even_value int NOT NULL );

CREATE OR REPLACE FUNCTION cf.cf_check( subject char(15) )
RETURNS char
AS $BODY$
DECLARE
  check_char char;
  odd_sum int := 0;
  even_sum int := 0;
  i int;
  current_value int;
  final_value int;
  odd_in text := '';
  even_in text := '';
  current_letter char;
BEGIN

  FOR i in 1..length( subject ) LOOP
      IF i % 2 = 0 THEN
         even_in := array_append( even_in,
                                  upper( substring( subject FROM i FOR 1 ) )::char );
      ELSE
        odd_in := array_append( odd_in,
                                upper( substring( subject FROM i FOR 1 ) )::char );
     END IF;
  END LOOP;


   SELECT sum(even_value)
   INTO even_sum
   FROM cf.check_chars
   JOIN unnest( even_in ) AS letter
   ON c = letter;

   SELECT sum(odd_value)
   INTO odd_sum
   FROM cf.check_chars
   JOIN unnest( odd_in ) AS letter
   ON c = letter;

   final_value := ( odd_sum + even_sum ) % 26;
   RAISE DEBUG 'cf_check: % + % %% 26 = %', odd_sum, even_sum, final_value;

  -- this is a trick: the remaining part
  -- indicates the positional order of the letter
  -- within the alphabet, which is
  -- the values into the table excluding digits
  SELECT c
  INTO STRICT check_char
  FROM cf.check_chars
  WHERE even_value = final_value
  AND c NOT IN ( '0', '1', '2', '3', '4', '5', '6', '7', '8', '9' );

  RETURN check_char;
END
$BODY$
LANGUAGE plpgsql;
```

In order to avoid querying data multiple times, that was my very first implementation, I placed the letters into an array and asked SQL to compute the *sum* for me.

The `plperl` version is shorter because of usage of postfix operators, even if it queries all the letters one by one.
Also usage of the *ellipsis* and `join` simplify the query construction:

```perl
CREATE OR REPLACE FUNCTION cf.cf_check( subject char(15) )
RETURNS char
AS $BODY$
  my ($subject) = @_;
  my ($odd_sum, $even_sum) = (0,0);
  my $index = 0;
  my $query;
  my $final_value;


  for ( split //, $subject ) {
    $index++;
    $query = sprintf( "SELECT odd_value, even_value FROM cf.check_chars WHERE c = '%s'", uc( $_ ) );
    $odd_sum  += spi_exec_query( $query, 1 )->{ rows }[ 0 ]->{ odd_value }  if ( $index % 2 != 0 );
    $even_sum += spi_exec_query( $query, 1 )->{ rows }[ 0 ]->{ even_value } if ( $index % 2 == 0 );
  }


  $final_value = ( $odd_sum + $even_sum ) % 26;
  elog( DEBUG, "cf_check: $subject -> $odd_sum + $even_sum % 26 = $final_value" );

  $query = sprintf( "SELECT c FROM cf.check_chars WHERE even_value = %d AND c NOT IN ( %s ) ",
                    $final_value,
                    join( ',', ( map { sprintf( "'%1s'", $_ ) } (0..9) ) ) );

  return spi_exec_query( $query, 1 )->{ rows }[ 0 ]->{ c };

END
$BODY$
LANGUAGE plperl;
```


