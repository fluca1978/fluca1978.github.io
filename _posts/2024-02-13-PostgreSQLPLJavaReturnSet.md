---
layout: post
title:  "Using PL/Java to Return SETOF RECORD"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pljava
permalink: /:year/:month/:day/:title.html
---
A simple way to return  multiple records from PL/Java

# Using PL/Java to Return SETOF RECORD

PL/Java allows a quite easy implementation of result set providers, objects that will produce rows that can be used as tables in queries.
In order to produce a result set, the main steps are:
1) implement the `ResultSetProvider` interface and its method to effectively produce the data;
2) build a PL/Java function that will instantiate the above `ResultsetProvider`, so that PL/Java will *wrap* such function into a `RETURN SETOF RECORD` SQL function.

In the following there is a quite simple demostration about the production of records from PL/Java.



## Implementing the `ResultSetProvider`

PL/Java has the `ResultSetProvider` interface that requires the implementation of two methods:
- `assignRowValues` that is called for every row in the result set, and must return `true` to indicate that a new row has been added to the result set, or `false` to indicate that the result set is complete and no more rows will be added;
- `close` that is called when the result set is closed by `assignRowValue`.

The `assignRowValues` function accepts two arguments:
- an `ResultSet` object that is the container for all the rows;
- a `long` value indicating the current row for which the method has been called. This counter *starts at zero*, as in normal Java list/array manipulations, not as in SQL.


Therefore, it is possible to implement the following methods as:

<br/>
<br/>
```java
public class Task1 implements ResultSetProvider {

    private final static Logger logger = Logger.getAnonymousLogger();

    public Task1( int maxRows ) {
		super();
		this.maxRows = maxRows;
    }

    private int maxRows = 10;


    @Override
    public boolean assignRowValues( ResultSet rs, int row )
	throws SQLException {

		if ( row > maxRows )
		    return false;

		logger.info( String.format( "Producing row %d/%d", row, maxRows ) );

		rs.updateString( 1, String.format( "Row %d out of %d from %s",
						   row,
						   maxRows,
						   this.getClass().getName() ) );
		rs.updateInt( 2, row );
		rs.updateInt( 3, maxRows );
		rs.updateDate( 4, new java.sql.Date( Calendar.getInstance().getTimeInMillis() ) );
		return true;

    }


    @Override
    public void close() {
		logger.info( "Closing resultset" );
    }
}


```
<br/>
<br/>

The `assignRowValues` function simply adds to the `ResultSet` a string field, two integers and one date field.
The production of the result set ends as soon as the produced rows count as in `maxRows` parameter, that is decided when the class is instantiated.


## Creating a function to call the producer

It is possible to create a PL/Java function that will instantiate the aboce class, returning it.
In order for PL/Java to understand that the function will produce a result set, the function must return a `ResultSetProvider`.

<br/>
<br/>
```java
    @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
    public static final ResultSetProvider rs_producer_pljava() throws SQLException {
		logger.log( Level.INFO, "Entering rs_producer_pljava" );

		Task1 producer = new Task1( 20 );
		return producer;
    }

```
<br/>
<br/>

Once the function has been compiled, and the JAR installed, there will be a function defined as:

<br/>
<br/>
```sql
testdb=> \sf rs_producer_pljava
CREATE OR REPLACE FUNCTION public.rs_producer_pljava()
 RETURNS SETOF record
 LANGUAGE java
 IMMUTABLE STRICT
AS $function$PWC256.Task1.rs_producer_pljava()$function$

```
<br/>
<br/>

Note how the function has been produced as `RETURN SETOF RECORD` and will call the PL(Java function, that in turn will instantiate the `ResultSetProviderr`.



## Using the function

It is now possible to query the function from SQL:

<br/>
<br/>
```sql
testdb=> select j.* from rs_producer_pljava() as j(t text, r int, m int, d date);
INFO:   PWC256.Task1 Entering rs_producer_pljava
INFO:   PWC256.Task1 Producing row 0/20
INFO:   PWC256.Task1 Producing row 1/20
INFO:   PWC256.Task1 Producing row 2/20
...
INFO:   PWC256.Task1 Producing row 18/20
INFO:   PWC256.Task1 Producing row 19/20
INFO:   PWC256.Task1 Producing row 20/20
INFO:   PWC256.Task1 Closing resultset

                t                 | r | m  |     d
-----------------------------------+---+----+------------
 Row 0 out of 20 from PWC256.Task1 | 0 | 20 | 2024-02-07
 Row 1 out of 20 from PWC256.Task1 | 1 | 20 | 2024-02-07
 Row 2 out of 20 from PWC256.Task1 | 2 | 20 | 2024-02-07
 Row 3 out of 20 from PWC256.Task1 | 3 | 20 | 2024-02-07
 Row 4 out of 20 from PWC256.Task1 | 4 | 20 | 2024-02-07
...

```
<br/>
<br/>

From the log messages it is possible to see that the result is being used to produce the records, and at the end it is closed.



## Passing dynamically the number of rows to produce

What if there is the need to decide dynamically how many rows the `ResultSetProvider` has to produce?
It simply requires to change the PL/Java function passing an integer argument:

<br/>
<br/>
```sql
    @Function( onNullInput = RETURNS_NULL, effects = IMMUTABLE )
    public static final ResultSetProvider rs_producer_pljava( int howManyRows ) throws SQLException {
		logger.log( Level.INFO, "Entering rs_producer_pljava" );

		if ( howManyRows <= 0 )
		    howManyRows = 5;

		Task1 producer = new Task1( howManyRows );
		return producer;
    }

```
<br/>
<br/>


And it is then possible to query the function with the following query:

<br/>
<br/>
```sql
testdb=> select j.* from rs_producer_pljava( 3 ) as j(t text, r int, m int, d date);
INFO:   PWC256.Task1 Entering rs_producer_pljava
INFO:   PWC256.Task1 Producing row 0/3
INFO:   PWC256.Task1 Producing row 1/3
INFO:   PWC256.Task1 Producing row 2/3
INFO:   PWC256.Task1 Producing row 3/3
INFO:   PWC256.Task1 Closing resultset
                t                 | r | m |     d
----------------------------------+---+---+------------
 Row 0 out of 3 from PWC256.Task1 | 0 | 3 | 2024-02-07
 Row 1 out of 3 from PWC256.Task1 | 1 | 3 | 2024-02-07
 Row 2 out of 3 from PWC256.Task1 | 2 | 3 | 2024-02-07
(2 rows)

```
<br/>
<br/>



# Conclusions

It is quite simple to use PL/Java to implement a row producer, even based on already existing code.
