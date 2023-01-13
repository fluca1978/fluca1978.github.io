---
layout: post
title:  "Perl Weekly Challenge 143: stealthing the grammars!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 143: stealthing the grammars!

It is sad that, after more than two years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 143](https://perlweeklychallenge.org/blog/perl-weekly-challenge-143/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)


<br/>
<br/>
And this week, as for the previous PWC, I had time to quickly implement the tasks also on PostgreSQL `plpgsql` language:
<br/>
- [Task 2 in plpgsql](#task2pg)





<a name="task1"></a>
## PWC 143 - Task 1

The first task was *hard*: implementing a calculator able to parses the main four operations and parenthesized expressions.
<br/>
I decided to implement it via grammars, as [well described here](https://andrewshitov.com/2018/10/31/creating-a-calculator-with-perl-6-grammars/){:target="_blank"}. In particular:
- I implemented a grammar to parse the expression;
- an action class was associated to the grammar to do the actual computation;
- the `MAIN` method does the whole application.

<br/>
Let's start from the application first:

<br/>
<br/>
```raku
sub MAIN( Str $expr ) {
    my $calculator = Calculator.parse( $expr, :actions( CalculatorActions ) );
    "{ $expr } = { $calculator.made }".say;
}
 
 ```
<br/>
<br/>

The end result is like the following:

<br/>
<br/>
```shell
$ raku ch-1.p6 "( 1 + 2 ) * 3 * 4 / 2"
( 1 + 2 ) * 3 * 4 / 2 = 18

```
<br/>
<br/>

Let's inspect the grammar first:

<br/>
<br/>
```raku
my Str $OPERATOR_ADD      = '+';
my Str $OPERATOR_MINUS    = '-';
my Str $OPERATOR_MULTIPLY = '*';
my Str $OPERATOR_DIVIDE   = '/';




grammar Calculator {
    rule TOP {
        ^ <expression> $
    }

    rule expression {
        | <operation>+ %% $<operator>=([$OPERATOR_ADD|$OPERATOR_MINUS])
        | <parenthesized-expression>
    }
    rule operation {
        <operand>+  %% $<operator>=([$OPERATOR_MULTIPLY|$OPERATOR_DIVIDE])
    }

    rule operand {
        | <number>
        | <parenthesized-expression>
    }

    rule parenthesized-expression {
        '(' <expression> ')'
    }

    token number { \d+ }
}

```

<br/>
<br/>

First of all, I define some "globals" to keep track of the operators, and then I have the grammar itself.
The `number` rules is simple: a number is just an integer value, therefore a bunch of digits. The next brick is the `operand`, that is either a number or an expression withint parentheses, represented by `parenthesized-expression`, which in turn is an `expression` between parentheses. An `operation` is an `operand` that is applied to a multiplication or a division, because these are the highest priority operators. An `expression` is then an `operation` applied to an addition or a subtraction (that have lower priority than `operation` itself) or another expression between parentheses.
<br/>
The action for the grammars looks like this:


<br/>
<br/>
```raku
class CalculatorActions {
    method TOP($/) {
        $/.make: $<expression>.made
    }


    method parenthesized-expression($/) {
        $/.make: $<expression>.made
    }

    method number($/) {
        $/.make: +$/
    }


    method operand($/) {
        $/.make: $<number> ?? $<number>.Int !! $<parenthesized-expression>.made;
    }


    # Computes a single operation in the form
    # a + b
    method do-compute( $left-operand, $operator, $right-operand ) {
        given $operator {
            when $OPERATOR_ADD      { $left-operand + $right-operand }
            when $OPERATOR_MINUS    { $left-operand - $right-operand }
            when $OPERATOR_MULTIPLY { $left-operand * $right-operand }
            when $OPERATOR_DIVIDE   { $left-operand / $right-operand }
        }
    }


    # Computes all the operation given the first operand, the set of operators
    # and the other operands.
    # For example:
    # 1 + 2 * 3
    # becomes
    # do-compute-all( 1, [+,*], [2,3])
    method do-compute-all( $left-operand is rw, @operators, @operands ) {
        while ( @operators.elems > 0 ) {
            $left-operand = self.do-compute( $left-operand,
                                             @operators.pop,
                                             @operands.pop );
        }
    }

    method operation($/) {
        # left part
        my $result = $<operand>[ 0 ].made;

        # if there is a right part ...
        if $<operator> {
            my @operators = $<operator>.map( *.Str );
            my @operands  = $<operand>[ 1..* ].map( *.made );

            self.do-compute-all( $result, @operators, @operands );
        }

        $/.make: $result;
    }



    method expression($/) {
        if $<parenthesized-expression> {
            $/.make: $<parenthesized-expression>.made
        }
        else {
            my $result = $<operation>[ 0 ].made;     

            if $<operator> {
                my @operators = $<operator>.map( *.Str );
                my @operands  = $<operation>[ 1..* ].map( *.made );

                self.do-compute-all( $result, @operators, @operands );

            }

            $/.make: $result;
        }
    }
}

```
<br/>
<br/>

There are some trivial methods, like `number` that converts a match into its integer form. The `do-compute` method does a single step operation, like `2 + 3`, while the `do-compute-all` does a whole expression.
In fact, both `expression` and `operation` calls the `do-compute-all` passing the first operand (in a mathematical sense) and the remaining operators and operands.




<a name="task2"></a>
## PWC 143 - Task 2

The second task was about finding out if a number was a *stealth* one, that is if does exist, given `$n`, that:
- `$a * $b = $c * $d = $n`
- `$a + $b = $c + $d + 1`.

<br/>
My implementation is as follows:

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 }, Bool :$verbose = False  ) {
    my @numbers = 1 ^..^ $n;

    # extract all the pairs to get the $n by multiplication
    my @pairs = @numbers.grep( $n %% * ).map( { $_, $n / $_, $_ + $n / $_ } );

    # now extract all the pairs couples that have a difference of one
    my $found = False;
    for 0 ..^ @pairs.elems -> $left {
        for $left ^..^ @pairs.elems -> $right {
            if @pairs[ $left ][ 2 ] - @pairs[ $right ][ 2 ] == any( 1, -1 ) {
                $found = True;
                "$n is stealth by { @pairs[ $left ][ 0..1 ].join( ',' ) } and { @pairs[ $right ][ 0..1 ].join( ',' ) }".say  if $verbose;
            }
        }
    }

    "1".say and exit if $found;
    "0".say;

}

```
<br/>
<br/>

In the beginning I search for all divisors of the given number, and map the result in an array with three elements: the divisor, the multiplier and the sum of the two. Then I do iterate other the resulting list of array to see if, with a nested loop, the current pair has a sum that differs exactly by `1` from the other pair.
<br/>
The rest is just usual printing stuff.
                      

<a name="task2pg"></a>
## PWC 143 - Task 2 in PostgreSQL

The PostgreSQL task 2 can be written as a poor translation of the Raku code:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION f_stealth( needle int )
  RETURNS int
AS $CODE$
  WITH numbers AS ( SELECT generate_series( 2, needle ) as n )
  , pairs AS ( 
    SELECT needle as n
           , n as divisor
           , needle / n as divisor_2
           , case needle % n
             when 0 then n + needle / n
             else null
             end as summix
      from numbers
  )
  , stealth as (
    select *
           , abs( summix - lag( summix, 1, summix )  over ( ORDER BY summix DESC ) ) as lag
           , abs( summix - lead( summix, 1, summix )  over ( ORDER BY summix DESC ) ) as lead
      FROM pairs
     where summix is not null
  )

  SELECT CASE count(*)
         WHEN 0 THEN 0
         ELSE 1
           END
  FROM stealth
  WHERE ( lag = 1 or lead = 1 );


  $CODE$
    LANGUAGE sql;

```
<br/>
<br/>

There is a *big* CTE that does all the trick:
- `numbers` generates the available numbers from `2` to `needle`;
- `pairs` produces all the pairs that are divisors of the `needle`, with their sum when it does make sense;
- `stealth` uses two window function that, on every row, gives the distance between the sum of the previous row and the following one;
- the final query selects only the tuple that have both a distance forward and backward of `1` (rows are ordered) and thus returns either `1` if there are rows, or `0` if there are not.
