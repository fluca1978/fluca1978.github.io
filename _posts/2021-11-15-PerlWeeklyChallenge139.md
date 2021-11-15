---
layout: post
title:  "Perl Weekly Challenge 139: repeating and sorting"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 139: repeating and sorting

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 139](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0139/){:target="_blank"}.

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
## PWC 139 - Task 1

The first task was really trivial, at least in its definition: do a *Jort Sort*, that is simply see if a given array of numbers is *already sorted*.
<br/>
Therefore, it just needs to compare the input with the sorted version of itself:

<br/>
<br/>
```raku
sub MAIN(  *@n where { @n.elems > 0  && @n.grep( * ~~ Int ).elems == @n.elems } ) {
    '1'.say and exit if @n ~~ @n.sort;
    '0'.say;
}

 ``` 
<br/>
<br/>

As you can see, it does suffice to *smart match* the `@n` array with its sorted version.
<br/>
Please note that there is literally much more code to do the input validation than to solve the task itself!




<a name="task2"></a>
## PWC 139 - Task 2

This task was about printing out the first five *long primes*, that are prime numbers where the decimal part is repeating with a size of the number less one.
<br/>
But Raku already has a `repeating-base` method in the `Rat` type, so the implementation is as short as:

<br/>
<br/>
```raku
sub MAIN( Int $limit where { $limit >= 1 }  = 5 ) {
    my @solutions;
    for 1 .. Inf {
        next if ! $_.is-prime;
        my $repeating-size = $_ - 1;
        my $repeating-part = ( 1 / $_ ).Rat.base-repeating( 10 )[ 1 ];
        next if ! $repeating-part;

        @solutions.push: $_ if $repeating-part.Str.chars == $repeating-size;
        last if @solutions.elems == $limit;
    }

    @solutions.join( ', ' ).say;
}


```
<br/>
<br/>

The idea is that I iterate on every integer number, and skip all those that are not already prime.
Then I get the `repeating-base` part, and skip all those numbers without a repeating part in the decimal part.
Then, if the repeating part has a textual length (i.e., a number of digits) that is `$repeating-size` (the current value minus one), the number is ok and is therefore added to the array of `@solutions`.
<br/>
If the array of `@solutions` is already filled with the required number of elements (in default five), the loop is terminated and the array is printed.





<a name="task1pg"></a>
## PWC 139 - Task 1 in PostgreSQL `plpgsql`

This implementation tried to emulate the Raku proposed one, but clearly there is no smart matching operator.
Therefore, I decided to convert an array into a string of text, and see if its textual representation is the same of the sorted array one.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
  f_jort_sort( n int[] )
  RETURNS int
AS $CODE$
  DECLARE
  n_ordered int[];
BEGIN
  SELECT ( ARRAY( SELECT unnest( n ) ORDER BY 1 ) )
    INTO n_ordered;

  IF array_to_string( n, '|' ) = array_to_string( n_ordered, '|' ) THEN
    RETURN 1;
  ELSE
    RETURN 0;
  END IF;
END
  $CODE$
  LANGUAGE plpgsql;
                   
```
<br/>
<br/>

In order to sort the array, I used a subquery to get it sorted by an `ORDER BY` clause over the `unnest` of the array.

<a name="task2pg"></a>
## PWC 139 - Task 2 in PostgreSQL `plpgsql`

This task has been really complex to implement in `plpgsql`, because of the lack of a clear way to find out the repeating base part.

<br/>
<br/>

``` plpgsql
CREATE OR REPLACE FUNCTION
  f_long_primes( l int default 5 )
  RETURNS SETOF bigint
AS $CODE$
  DECLARE
  current_value numeric;
  i int;
  is_prime bool;
  rational_part text;
  done int := 0;
  repeating_part_size int := 0;
  repeating_part      text;
  result_as_text      text;
  text_array          text[];

BEGIN

  for current_value IN 1 .. 99999 LOOP
    is_prime := true;

    -- check if this is prime
    for i in 2 .. ( current_value - 1 ) LOOP
      if current_value % i = 0 THEN
        is_prime := false;
        exit; -- terminate this loop
        END IF;
    END LOOP;

    -- avoid inspecting this number if it is not prime
    if is_prime = false THEN
      continue;
    END IF;

    result_as_text := ( 1 / current_value::numeric )::text;
    i := strpos( result_as_text, '.' );
    rational_part := substr( result_as_text, i + 1 );
    repeating_part_size :=  current_value - 1;
    repeating_part      := substr( rational_part, 1, repeating_part_size );

    RAISE DEBUG 'Inspecting % -> % {%} => % ( %, % )',
      current_value,
      rational_part,
      repeating_part_size,
      repeating_part,
      i,
      ( 1 / current_value )::text;


    text_array := regexp_split_to_array( rational_part, repeating_part  );
    is_prime   := true;
    FOREACH result_as_text IN ARRAY text_array LOOP
      IF result_as_text <> '' AND substr( repeating_part, 1, length( result_as_text ) ) <> result_as_text THEN
        is_prime := false;
        exit;
      END IF;
    END LOOP;

    IF is_prime THEN
      RETURN NEXT current_value;
      done := done + 1;
    END IF;

    IF done > l THEN
      EXIT;
    END IF;
  END LOOP;

  RETURN;
END
  $CODE$
  LANGUAGE plpgsql;
                                                                         

```
<br/>
<br/>


The idea is as follows:
- I do loop to geenrate all integers and check if the number is prime, otherwise I cannot continue with this value;
- once I tested the number is prime, I do get its `numeric` division, so to get the max precision available;
- I split the string representation of the division to keep the `rational_part` and then I extract the first digits for the `repeating_part`;
- now the complex part is to check if the part is repeating, so I use `regexp_split_to_array` of the decimal part and the repeating part. If the `repeating_part` matches, the array element will be an empty string, so I have to check every element of the regexp matching array to see if it is an empty string. There is, however, a critical case, that is if the number of digits terminates before having composed all the repeating part, so I simply checks if the last element of the array is  within the repeating part string.
<br/>
In the end, I do return a number and terminate everything if the returned numbers collection is of the right size.
