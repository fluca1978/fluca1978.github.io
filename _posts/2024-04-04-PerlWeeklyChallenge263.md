---
layout: post
title:  "Perl Weekly Challenge 263: iterating and filtering arrays"
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

# Perl Weekly Challenge 263: iterating and filtering arrays

This post presents my solutions to the [Perl Weekly Challenge 263](https://perlweeklychallenge.org/blog/perl-weekly-challenge-263/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 263 - Task 1 - Raku](#task1)
- [PWC 263 - Task 2 - Raku](#task2)
- [PWC 263 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 263 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 263 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 263 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 263 - Task 1 in PL/Java](#task1pljava)
- [PWC 263 - Task 2 in PL/Java](#task2pljava)
- [PWC 263 - Task 1 in Python](#task1python)
- [PWC 263 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 263 - Task 1 - Raku Implementation

The first task was to get all the indexes of a given array where the content of the indexed cell has the value equal to a given one.

<br/>
<br/>
```raku
sub MAIN( Int $k, *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    @nums.sort.grep( * ~~ $k, :k ).say;
}

```
<br/>
<br/>

Thanks to the fact that `grep` can return the indexes, this task can be solved in a single line.


<a name="task2"></a>
## PWC 263 - Task 2 - Raku Implementation

The second task was about merging two arrays made by pairs, where the first element in each pair identifies an item and the second its quantity.

<br/>
<br/>
```raku
sub MAIN() {
    my $items1 = [ [1,1], [2,1], [3,2] ];
    my $items2 = [ [2,2], [1,3] ];

    my %quantities;
    %quantities{ $_[ 0 ] } += $_[ 1 ] for $items1.flat;
    %quantities{ $_[ 0 ] } += $_[ 1 ] for $items2.flat;

    %quantities.Array.say;
}

```
<br/>
<br/>
The `%quantities` hash is keyed by the items, and thanks to autovivification I can simply sum the value with the new one in the array.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 263 - Task 1 - PL/Perl Implementation

With a combination of `grep`, `map` and `sort` this can be sovled in a single instruction.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc263.task1_plperl( int, int[] )
RETURNS SETOF int
AS $CODE$

   my ( $k, $nums ) = @_;

   return [
   map { $_->[ 0 ] }
   grep { $_->[ 1 ] == $k }
   map { [ $index++, $_ ] }
   sort $nums->@* ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The idea is to `map` the element of the array with a pair with its index, then to `grep` to find a matching element and to `map` again to get only the index.


<a name="task2plperl"></a>
## PWC 263 - Task 2 - PL/Perl Implementation

The idea is similar to that of Raku: use an hash `$q` to store the items and their quantity, and then return all the results.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc263.task2_plperl( int[], int[] )
RETURNS TABLE( item int, qty int )
AS $CODE$

   my ( $items1, $items2 ) = @_;
   my $q = {};

   for my $pair ( $items1->@*, $items2->@* ) {
       $q->{ $pair->[ 0 ] } += $pair->[ 1 ];
   }


   return_next( { item => $_, qty => $q->{ $_ } } )  for ( sort keys $q->%* );
   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 263 - Task 1 - PL/PgSQL Implementation

A single query does suffice: thanks to `row_number()` it is possible to get a kind of autoincrement value for each tuple, and then it is possible to filter on the matching.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc263.task1_plpgsql( k int, nums int[] )
RETURNS SETOF int
AS $CODE$

   SELECT x
   FROM (
   	SELECT v, row_number() over ( order by v ) as x
	FROM unnest( nums ) v
	WHERE v = k
	)
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 263 - Task 2 - PL/PgSQL Implementation

A much more verbose approach.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc263.task2_plpgsql( i int[], j int[] )
RETURNS TABLE( item int, qty int )
AS $CODE$
DECLARE

BEGIN

	CREATE TEMPORARY TABLE IF NOT EXISTS q( item int, qty int );
	TRUNCATE TABLE q;

	FOR x IN 1 .. array_length( i, 1 ) LOOP
	    IF mod( x, 2 ) = 0 THEN
	       CONTINUE;
	    END IF;
	    INSERT INTO q( item, qty )
	    VALUES ( i[ x ], i[ x + 1 ] );

	END LOOP;

	FOR x IN 1 .. array_length( j, 1 ) LOOP
	     IF mod( x, 2 ) = 0 THEN
	       CONTINUE;
	    END IF;
	    INSERT INTO q( item, qty )
	    VALUES ( j[ x ], j[ x + 1 ] );

	END LOOP;

	RETURN QUERY
	SELECT q.item, sum( q.qty )::int
	FROM q
	GROUP BY q.item;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


I use a temporay table `q` as an hashtable, where I store every item with its quantity. In the end, I `GROUP BY` the item identifier and `SUM` the quantity.


# Java Implementations

<a name="task1pljava"></a>
## PWC 263 - Task 1 - PostgreSQL PL/Java Implementation

There is much more code to convert arrays into lists and viceversa!

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc263",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task1_pljava( int k, int[] nums ) throws SQLException {
	logger.log( Level.INFO, "Entering pwc263.task1_pljava" );

	List<Integer> source = new LinkedList<Integer>();
	for ( int v : nums )
	    source.add( v );

	Collections.sort( source );

	final List<Integer> result = new LinkedList<Integer>();
		IntStream.range( 0, source.size() )
			.forEach( index -> {
				if ( source.get( index ) == k )
 					result.add( index );
			} );

	   int res[] = new int[ result.size() ];
	   for ( int i = 0; i < res.length; i++ )
          res[ i ] = result.get( i );

	    return res;

    }
}

```
<br/>
<br/>

The idea is to use *streams* to iterate over the `nums` (converted into a list named `source`) to append every index to the `result` list, that is then returned as an array of integers.


<a name="task2pljava"></a>
## PWC 263 - Task 2 - PostgreSQL PL/Java Implementation

A much more verbose approach.


<br/>
<br/>
```java
public class Task2 implements ResultSetProvider {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc263",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final ResultSetProvider task2_pljava( int i[], int j[] ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc263.task2_pljava" );
		return new Task2( i, j );
    }


    Map<Integer, Integer> quantity = new HashMap<Integer, Integer>();
    List<Integer> items = new LinkedList<Integer>();

    public Task2( int[] i, int[] j ) {
		super();

		int index = 0;
		while ( index < i.length ) {
		    int item = i[ index ];
		    int qty  = i[ index + 1 ];
		    quantity.put( i[ index ], i[ index + 1 ] );
		    index += 2;
		}

		index = 0;
		while ( index < j.length ) {
		    int item = j[ index ];
		    int qty  = j[ index + 1 ];
		    if ( quantity.containsKey( item ) )
			qty += quantity.get( item );


		    quantity.put( item, qty );
		    index += 2;
		}

		items.addAll( quantity.keySet() );

    }

    @Override
    public boolean assignRowValues( ResultSet tuples, int row )
	throws SQLException {

		if ( items.isEmpty() || quantity.isEmpty() )
		    return false;

		int item = items.remove( 0 );

		tuples.updateInt( 1, item );
		tuples.updateInt( 2, quantity.get( item ) );
		return true;
    }

    @Override
    public void close() {}
}

```
<br/>
<br/>


The idea is to return a result set made by two integer columns, therefore the class implements a `ResultSetProvider`. Internally, the `quantity` hashmap is used to store the items and their quantity, and the `keys` list is used to pull one key at a time when PostgreSQL requires a new row. in fact, the `row` coulb be diferent from the item identifier.


# Python Implementations

<a name="task1python"></a>
## PWC 263 - Task 1 - Python Implementation

A solution inspired by Perl: use `filter` to extract only the indexes that make a match.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    k      = int( args[ 0 ] )
    nums   = list( map( int, args[ 1: ] ) )
    nums.sort()
    return list( filter( lambda x: nums[ x ] == k, range( 0, len( nums ) ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 263 - Task 2 - Python Implementation

A similar approach to the Java implementation: use a dictionary as an hash table to store every item with its quantity.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    quantity = {}
    for index in range( 0, len( args ) - 1 ):
        if index % 2 == 1:
            continue

        item = args[ index ]
        qty  = int( args[ index + 1 ] )
        if item in quantity:
            quantity[ item ] += qty
        else:
            quantity[ item ] = qty

    return quantity


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
