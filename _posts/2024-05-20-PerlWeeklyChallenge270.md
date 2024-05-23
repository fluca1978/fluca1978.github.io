---
layout: post
title:  "Perl Weekly Challenge 270: no passion this week!"
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

# Perl Weekly Challenge 270: no passion this week!

This post presents my solutions to the [Perl Weekly Challenge 270](https://perlweeklychallenge.org/blog/perl-weekly-challenge-270/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 270 - Task 1 - Raku](#task1)
- [PWC 270 - Task 2 - Raku](#task2)
- [PWC 270 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 270 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 270 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 270 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 270 - Task 1 in PL/Java](#task1pljava)
- [PWC 270 - Task 2 in PL/Java](#task2pljava)
- [PWC 270 - Task 1 in Python](#task1python)
- [PWC 270 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 270 - Task 1 - Raku Implementation

The first task was about computing the positions (indexes) of `1` in a matrix, assuming a *good one* must appear only on a row and column.

<br/>
<br/>
```raku
sub MAIN() {
    my @matrix =  [1, 0, 0],
                  [0, 0, 1],
                  [1, 0, 0],
    ;

    my @ones;
    for 0 ..^ @matrix.elems -> $row {
		@ones.push: @matrix[ $row ].grep( { $_ == 1 }, :k ).map( { $row, $_, '%02d-%02d'.sprintf( $row, $_ )  } ).flat;
    }


    for @ones -> $current_one {
		$current_one[ 0, 1 ].join( ',' ).say
		    if @ones.grep( {  $current_one[ 2 ] ne $_[ 2 ]
			       && ( $_[ 1 ] == $current_one[ 1 ] || $_[ 0 ] == $current_one[ 0 ] ) } ).elems == 0
    }

}

```
<br/>
<br/>

The idea is to compute all the indexes of `1` found in the matrix, stored into `@ones` with the column, the row and an identifier.
Then I process all the results, ensuring there is not another element with the same identifier in the matrix, in such case the position indicates a good one to display.

<a name="task2"></a>
## PWC 270 - Task 2 - Raku Implementation

The second task was about computing the *score* to modify an array of given integers so that all the integers become equal to the max value within the array. At each step, I can choose to add `1` unit to a single element or a couple, and depending on the operation I have two different input costs.

It is not clear to me if I have to compute the *minimum score*, I haven't implemented this logic.

<br/>
<br/>
```raku
sub MAIN( Int $single,
	  Int $double,
	  *@nums is copy where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {

    my $current_max = @nums.max;
    my @need_operation = @nums.grep( * < $current_max, :k );
    my $score = 0;

    while ( @need_operation ) {

		if @need_operation.elems == 1 {
		    # single operation
		    @nums[ @need_operation[ 0 ] ] += 1;
		    $score += $single;
		}
		elsif @need_operation.elems > 1 {
		    @nums[ @need_operation[ 0 ] ] += 1;
		    @nums[ @need_operation[ 1 ] ] += 1;
		    $score += $double;
		}

		@need_operation = @nums.grep( * < $current_max, :k );
    }

    $score.say;

}

```
<br/>
<br/>

My implementation is quite simple: I compute the maximum value in the array, and then extract all the elements that are not at the same value.
Then, while I've more than one element, I apply the double operation, and as last, a single operation.

In the case there is to compute the minimum score, not implemented in the above nor in the following solutions, the two costs have to be compared and it is required to decide which operatin costs less to make two changes.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 270 - Task 1 - PL/Perl Implementation

Same approach as in Raku, returning a table with the found positions as `c` column and `r` row.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc270.task1_plperl( int[][] )
RETURNS TABLE( r int, c int )
AS $CODE$

   my ( $matrix ) = @_;
   my @pnes;

   for my $row ( 0 .. $matrix->@* -1 ) {
       for my $column ( 0 .. $matrix->@[ $row ]->@* -1 ) {
       	   next if ( $matrix->@[ $row ]->@[ $column ] != 1 );
	   push @ones, [ $row, $column, sprintf( '%02d-%02d', $row, $column ) ];
       }
   }

   for my $position ( @ones ) {
       if ( grep( { $position->[ 2 ] != $_->[ 2] && ( $position->[ 0 ] == $_->[ 0 ] || $position->[ 1 ] == $_->[ 1 ] ) } @ones ) == 0 ) {
       	  return_next( { r => $position->@[ 0 ], c => $position->@[ 1 ] } );
       }
   }


   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 270 - Task 2 - PL/Perl Implementation

Same approach as in Raku, a little more verbose because I implement the inner function `max`.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc270.task2_plperl( int, int, int[] )
RETURNS int
AS $CODE$

   my ( $single, $double, $nums ) = @_;
   my $score = 0;

   my $max = sub {
      my ( $nums ) = @_;
      my $value = undef;

      for ( $nums->@* ) {
      	  $value = $_ if ( ! $value || $value < $_ );
      }

      return $value;
   };

   my $current_max = $max->( $nums );
   my @need_operation;


   for ( 0 .. $nums->@* - 1 ) {
       push @need_operation, $_ if ( $nums->@[ $_ ] < $current_max );
   }

   while ( @need_operation > 0 ) {
      if ( @need_operation > 1 ) {
         $nums->[ $need_operation[ 0 ] ]++;
         $nums->[ $need_operation[ 1 ] ]++;
         $score += $double;
      }
      elsif ( @need_operation == 1 ) {
         $nums->[ $need_operation[ 0 ] ]++;
         $score += $single;
      }

      @need_operation = ();
      for ( 0 .. $nums->@* - 1 ) {
          push @need_operation, $_ if ( $nums->@[ $_ ] < $current_max );
      }
   }

   return $score;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 270 - Task 1 - PL/PgSQL Implementation

I cheated, I use PL/Perl here.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc270.task1_plpgsql( matrix int[][] )
RETURNS SETOF int[]
AS $CODE$
   SELECT pwc270.task1_plperl( matrix );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 270 - Task 2 - PL/PgSQL Implementation

Again, this week I was not really inspired to do the solutions in SQL, so I call PL/Perl.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc270.task2_plpgsql( s int, d int, nums int[] )
RETURNS int
AS $CODE$
   SELECT pwc270.task2_plperl( s, d, nums );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 270 - Task 1 - PostgreSQL PL/Java Implementation

The approach is similar to the Raku one, but much more verbose because I was running out fantasy!


<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc270",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final String[] task1_pljava( int row_size, int[] matrix ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc270.task1_pljava" );

		List<Integer> rows = new LinkedList<Integer>();
		List<Integer> cols = new LinkedList<Integer>();
		List<String> result = new LinkedList<String>();

		// seek for ones
		for( int row = 0; row < ( matrix.length / row_size ) ; row++ )
		    for ( int col = 0; col < row_size; col++ )
			if ( matrix[ row * row_size  + col ] ==  1 ) {
			    rows.add( row  );
			    cols.add( col );
			}

		for ( int i = 0; i < rows.size(); i++ ) {
		    int current_row = rows.get( i );
		    int current_col = rows.get( i );
		    boolean ok      = true;

		    for ( int r = 0; r < rows.size(); r++ )
				if ( r == i )
				    continue;
				else if ( rows.get( r ) == current_row )
				    ok = false;

		    for ( int c = 0; c < cols.size(); c++ )
				if ( c == i )
				    continue;
				else if ( cols.get( c ) == current_col )
				    ok = false;


		    if ( ok )
			result.add( String.format( "Row %d Col %d", current_row, current_col ) );
		}

		String[] to_return = new String[ result.size() ];
		int i = 0;
		for ( String s : result )
		    to_return[ i++ ] = s;

		return to_return;

    }
}

```
<br/>
<br/>


I use two different lists, of the same size, to contain the rows and the columns of the `1` elements found.
Then I iterate over the found elements to ensure there are no other ones on the same row or row.
Last, I return a string with the specified position of the element. The need to return a `String[]` is simply to avoid to implement a `ResultSetProvider`.

<a name="task2pljava"></a>
## PWC 270 - Task 2 - PostgreSQL PL/Java Implementation


<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc270",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int task2_pljava( int score_single, int score_double, int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc270.task2_pljava" );

		int score = 0;
		List<Integer> nums = Arrays.stream( numbers ).boxed().collect( Collectors.toList() );

		// get the max
		Integer max = nums.stream().max( Integer::compare ).get();

		List<Integer> needs_operation = new LinkedList<Integer>();
		do {
		    needs_operation.clear();
		    IntStream
			.range( 0, nums.size() - 1 )
			.forEach( index -> {
				if ( nums.get( index ) < max )
				    needs_operation.add( index );
			    } );


		    if ( needs_operation.size() == 1 ) {
				score += score_single;
				int index = needs_operation.get( 0 );
				int value = nums.get( index );
				nums.set( index, ++value );
		    }
		    else if ( needs_operation.size() >= 2 ) {
				score += score_double;

				for ( int i = 0; i < 2; i ++ ) {
				    int index = needs_operation.get( i );
				    int value = nums.get( index );
				    nums.set( index, ++value );
				}
		    }

		} while ( ! needs_operation.isEmpty() );

		return score;

    }
}

```
<br/>
<br/>


The idea is always the same as in the other approaches, but this time I use the *stream* API to get the indexes of the elements that require to be incremented. Then I perform the operation and compute again which elements are still not at the maximum level.


# Python Implementations

<a name="task1python"></a>
## PWC 270 - Task 1 - Python Implementation

The idea is always the same.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    row_size = int( args[ 0 ] )
    matrix   = []

    # build the matrix
    current_row = 0
    matrix.append( [] )
    for x in args[ 1: ]:

        if len( matrix[ current_row ] ) >= row_size:
            current_row += 1
            matrix.append( [] )

        matrix[ current_row ].append( int( x ) )


    ones = []
    for r in range( 0, len( matrix ) ):
        for c in range( 0, len( matrix[ r ] ) ):
            if matrix[ r ][ c ] == 1:
                ones.append( [ r, c ] )

    for current in ones:
        found = list( filter( lambda x: x[ 0 ] == current[ 0 ] or x[ 1 ] == current[ 1 ], ones ) )
        if len( found ) == 1:
            print( current )


    return ""

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>


The only difference with the other implementations is that I check that the list of elements found with a `1` on a given row/column must be of size '1` to indicate the only element itself.

<a name="task2python"></a>
## PWC 270 - Task 2 - Python Implementation


Same implementation as in the other cases.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    single = int( args[ 0 ] )
    double = int( args[ 1 ] )
    nums   = list( map( int, args[ 2: ] ) )

    # compute the current max
    max = None
    for v in nums:
        if not max or v > max:
            max = v

    need_operation = list( filter( lambda x: nums[ x ] < max, range( 0, len( nums ) ) ) )
    score = 0
    while len( need_operation ) > 0:
        if len( need_operation ) > 1 :
            score += double
            nums[ need_operation[ 0 ] ] += 1
            nums[ need_operation[ 1 ] ] += 1
            need_operation = need_operation[ 2: ]
        elif len( need_operation ) == 1:
            score += single
            nums[ need_operation[ 0 ] ] += 1
            need_operation = need_operation[ 2: ]


    return score



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
