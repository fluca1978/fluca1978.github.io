---
layout: post
title:  "Perl Weekly Challenge 257: Post-Valentine Challenge"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 257: Post-Valentine Challenge

This post presents my solutions to the [Perl Weekly Challenge 257](https://perlweeklychallenge.org/blog/perl-weekly-challenge-257/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 257 - Task 1 - Raku](#task1)
- [PWC 257 - Task 2 - Raku](#task2)
- [PWC 257 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 257 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 257 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 257 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 257 - Task 1 in PL/Java](#task1pljava)
- [PWC 257 - Task 2 in PL/Java](#task2pljava)
- [PWC 257 - Task 1 in Python](#task1python)
- [PWC 257 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 257 - Task 1 - Raku Implementation

The first task was to remap an array of integers into an array where every value was the count of elements of the array lower than the current element value. This can be sovled with a single line in Raku:

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.elems == @numbers.grep( * ~~ Int ).elems } ) {
    @numbers.map( -> $current { @numbers.grep( * < $current ).elems } ).join( ', ' ).say;
}
```
<br/>
<br/>



<a name="task2"></a>
## PWC 257 - Task 2 - Raku Implementation

Given a matrix of integers, find out if the matrix can be reduced according to specific rules.
The rules are not so clear to me, but essentially:
- every row must be either full of zero or the leftmost not zero element must be a one;
- every leftmost one in the row is called a *leading one* and must be rightmost of the leading ones of the above rows;
- the zero rows must be all at the bottom of the matrix.

<br/>
<br/>
```perl6
sub MAIN() {
    my $M = [
        [1, 1, 0],
        [0, 1, 0],
        [0, 0, 0]
    ];

    my $ok = True;
    my @zero-rows;
    my @leadings;
    for 0 ..^ $M.elems -> $row {
		my $first-in-row = Nil;


		for 0 ..^ $M[ $row ].elems -> $col {
		    next if $M[ $row ][ $col ] == 0;
		    next if $first-in-row;

		    $first-in-row = $M[ $row ][ $col ] if ! $first-in-row;

		    if ( $first-in-row == 1 ) {
				# leading one
				@leadings.push: { row => $row, col => $col };
				# if not other leadings, skip any check
				next if @leadings.elems <= 1;

				$ok = False and last if ( @leadings[ * - 2 ]<col> >= $col && @leadings[ * - 2 ]<row> != ( $row - 1 ) );
		    }
		}

		if ! $first-in-row {
		    # the row was made by all zeros
		    @zero-rows.push: $row;
		    # if this is the first zero row, cannot check anything
		    next if @zero-rows.elems <= 1;
		    # this is not the first zero row, so the previous row must be also zero!
		    $ok = False if @zero-rows[ * - 2 ] != ( $row - 1 );
		}
		elsif $first-in-row != 1 {
		    $ok = False;
		}

		'0'.say and exit if ( ! $ok );
    }

    '1'.say; # here everything is fine
}

```
<br/>
<br/>

I iterate on every row and column, and keep a `first-in-row` element that is valued only with the leftmost non zero value.
Then, if such element is a one, it is also pushed to a `@leadings` array that keeps track of the rows with the leading values, storing a `Pair` with the row and column index. On the other hand, if the `first-in-row` is not valued, the row is filled by zeros, so append its index to the `@zero-rows` arraay.
Then, I check if the `@zero-rows` array if consistent, as well as columns in `@leadings` go from left to right.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 257 - Task 1 - PL/Perl Implementation

Really simple implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc257.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $numbers ) = @_;

   for my $current ( $numbers->@* ) {
       return_next( scalar( grep( { $_ < $current } $numbers->@* ) ) );
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 257 - Task 2 - PL/Perl Implementation

The implement follows the same reasoning of that in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc257.task2_plperl( int[][] )
RETURNS bool
AS $CODE$

   my ( $matrix ) = @_;

   my $ok = 1;
   my @zero_rows;
   my @leadings;

 my $row ( 0 .. $matrix->@* - 1 ) {
       my $first_in_row = undef;

       for my $col ( 0 .. $matrix->@* - 1 ) {
       	   next if $matrix->[ $row ][ $col ] == 0;
	   next if $first_in_row;

	   $first_in_row = $matrix->[ $row ][ $col ] if ( ! $first_in_row );

	   # the first non-zero value of every row must be a one
	   $ok = 0 and last if ( $first_in_row != 1 );

           # leading one
	   push @leadings, { row => $row, col => $col };

       }

       if ( ! $first_in_row ) {
       	  # the row was filled with zeros
	  push @zero_rows, $row;

	  next if @zero_rows <= 1;
	  $ok = 0 and last if ( @zero_rows[ -1 ] != ( $row - 1 ) );
       }
       elsif ( $first_in_row == 1 ) {
       	   next if @leadings <= 1;
	   $ok = 0 and last if ( @leadings[ -2 ]->{ col } >= @leadings[ -1 ]->{ col }
	                      || @leadings[ -2 ]->{ row } != @leadings[ -1 ]->{ row } - 1 );
       }

       return $ok if ! $ok;
   }

   return $ok;
   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 257 - Task 1 - PL/PgSQL Implementation

This can be sovled with a single query.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc257.task1_plpgsql( numbers int[] )
RETURNS SETOF int
AS $CODE$
   select z
   from unnest( numbers ) v,
        lateral ( select count(x)
	          from unnest( numbers ) x
		  WHERE x < v ) as y(z);
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

Note the usage of **lateral** join to evaluate the current `v` element for every subquery.


<a name="task2plpgsql"></a>
## PWC 257 - Task 2 - PL/PgSQL Implementation

Here I cheat, and use the PL/Perl implementation directly, because PL/PgSQL is really poor at handling matrixes.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc257.task2_plpgsql( matrix int[] )
RETURNS bool
AS $CODE$
	select pwc257.task2_plperl( matrix );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 257 - Task 1 - PostgreSQL PL/Java Implementation

The task needs to implement a `ResultSetProvider` since the function must return a single row for every evaluation.

<br/>
<br/>
```java
public class Task1 implements ResultSetProvider {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc257",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final ResultSetProvider task1_pljava( int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering task1_pljava" );
		return new Task1( numbers );

    }

    public Task1( int[] nums ) {
		numbers = nums;
    }

    private int[] numbers;

    @Override
    public boolean assignRowValues(ResultSet receiver, int currentRow)
	throws SQLException {

		if ( currentRow >= numbers.length )
		    return false;

		int current = numbers[ currentRow  ];

		int count = 0;
		for ( int other : numbers )
		    if ( other < current )
			count++;

		receiver.updateInt( 1, count );
		return true;

    }



    @Override
    public void close() { }

}

```
<br/>
<br/>

The idea is that the function will create a `Task1` object that will store the input array of numbers, and then will produce every row thru the `assignRowValues()` function. This function, in turn, counts the number of elements less than the current one, dependent on the `currentRow` passed by the result set provider.


<a name="task2pljava"></a>
## PWC 257 - Task 2 - PostgreSQL PL/Java Implementation

Here there is a complication: PL/Java cannot handle an `int[][]` as a matrix, thus I use the same trick I would do with PL/PgSQL, hence passing a linear array and a column size to split the linear array into a matrix.

<br/>
<br/>
```java
public class Task2 {
    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc257",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final boolean task2_pljava( int[] matrix, int cols  ) throws SQLException {
		logger.log( Level.INFO, "Entering task2_pljava" );

		boolean ok = false;
		List<Integer> zero_rows = new LinkedList<Integer>();
		Integer current = null;
		List<int[]> leadings = new LinkedList<int[]>();

		for ( int row = 0; row < matrix.length / cols ; row++ ) {
		    current = null;

		    for ( int col = 0; col < cols; col++ ) {

			int element = matrix[ row * cols + col ];
			if ( element == 0 || current != null )
			    continue;

			current = element;
			if ( current != 1 )
			    return false;

			leadings.add( new int[]{ row, col } );

			if ( leadings.size() <= 1 )
			    continue;

			if ( leadings.get( leadings.size() - 2 )[ 0 ] == ( leadings.get( leadings.size() - 1 )[ 0 ] - 1 )
			     && leadings.get( leadings.size() - 2 )[ 1 ] >= leadings.get( leadings.size() - 1 )[ 1 ] )
			    return false;
		    }

		    if ( current == null ) {
			// all zero row
			zero_rows.add( row );

			if ( zero_rows.size() <= 1 )
			    continue;

			if ( zero_rows.get( zero_rows.size() - 2 ) != ( zero_rows.get( zero_rows.size() - 1) - 1 ) )
			    return false;
		    }
		}

		return true;

    }
}

```
<br/>
<br/>

The logic is the same used in PL/Perl and Raku.

# Python Implementations

<a name="task1python"></a>
## PWC 257 - Task 1 - Python Implementation

Same implementation as PL/Perl, with a lot more clutter to convert results into lists.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    output = []
    numbers = list( map( int, args ) )
    for current in numbers:
        output.append( len( list( filter( lambda x: x < current, numbers ) ) ) )

    return output


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 257 - Task 2 - Python Implementation

The same logic in all the other implementations, with the usage of a dictionary instead of an hash.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( numbers ):

    ok = False
    zero_rows = []
    leadings  = []

    for row in range( 0, len( numbers ) ):
        first_in_row = None

        for col in range( 0, len( numbers[ row ] ) ):
            if numbers[ row ][ col ] == 0:
                continue

            if first_in_row != None:
                continue

            first_in_row = numbers[ row ][ col ]


            if first_in_row != 1:
                return False

            leadings.append( { 'row' : row, 'col' : col } )
            if len(leadings) <= 1:
                continue

            if leadings[ -2 ][ 'row' ] != ( row - 1 ) and leadings[ -2 ][ 'col' ] >= col:
                return False

        if first_in_row == None:
            zero_rows.append( row )

            if len( zero_rows ) <= 1:
                continue
            if zero_rows[ -2 ] != ( row - 1 ):
                return False
        elif first_in_row == 1:
            if len( leadings ) <= 1:
                continue
            if leadings[ -2 ][ 'row' ] != leadings[ -1 ][ 'row' ] -1 and leadings[ -2 ][ 'col' ] >= leadings[ -1 ][ 'col' ]:
                return False

    return True


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( [ [0,1,0], [0,0,1], [0,0,0] ] ) )

```
<br/>
<br/>
