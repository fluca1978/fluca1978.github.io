---
layout: post
title:  "Statements with RETURNING: Perl and Java clients"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- java
- perl
permalink: /:year/:month/:day/:title.html
---
PostgreSQL statements support the ~RETURNING~ predicate that allows a statement that manipulates tuples to return a set of column of such tuples. It is easy to use such statements on a client-basis to get back data not available when the query has been written.

# Statements with RETURNING: Perl and Java clients

Statements such as `INSERT`, `DELETE` and `UPDATE` can have a `RETURNING` predicate that allows to get back the data that the statement has manipulated. On a theoretical point of view, it is like the following two statements are executed:

```sql
INSERT|UPDATE|DELETE tuples;
SELECT above_tuples;
```

From within the database connection, such `RETURNING` statement can be very useful to *see* which tuples have been modified, and from a client perspective it can be used to get random and serial-based data.
Consider a simple table defined as follows:

```sql
CREATE TABLE foo
( 
  pk serial
  , rv float
);
```

and consider the following simple statement to insert values:

```sql
INSERT INTO foo( rv )
SELECT random()
FROM generate_series( 1, 10 );
```

The above query inserts 10 tuples with `rv` set to a random value and `pk` set to the next value of the associated sequence. In other words, it is not possible to know in advance what values have been inserted.

Thanks to `RETURNING` this knowledge is pushed back to the client, and can be consumed as a normal result set, that means as if the client issued a `SELECT` statement.

As a simple example, consider [the following Perl client](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/clients/perl/returning.pl):

```perl
use DBI;
use v5.20;

my $dbh = DBI->connect("dbi:Pg:dbname=testdb;host=localhost;port=5432",
                       'luca',
                       '',
                       {AutoCommit => 0} );
my $query = <<'END_QUERY';
INSERT INTO foo( rv )
SELECT random()
FROM generate_series( 1, 10 )
RETURNING pk, rv;
END_QUERY

my $statement = $dbh->prepare( $query );
$statement->execute();
while ( my $result = $statement->fetchrow_hashref ) {
    say sprintf 'The statement inserted pk = %d and a random value rv = %f',
        $result->{ pk },
        $result->{ rv };
}


$dbh->disconnect();
```

that produces an output similar to the following one:

```sh
The statement inserted pk = 11 and a random value rv = 0.258626
The statement inserted pk = 12 and a random value rv = 0.877215
The statement inserted pk = 13 and a random value rv = 0.900430
The statement inserted pk = 14 and a random value rv = 0.312273
The statement inserted pk = 15 and a random value rv = 0.300636
The statement inserted pk = 16 and a random value rv = 0.401800
The statement inserted pk = 17 and a random value rv = 0.446666
The statement inserted pk = 18 and a random value rv = 0.352235
The statement inserted pk = 19 and a random value rv = 0.390648
The statement inserted pk = 20 and a random value rv = 0.790937
```

and the [corresponding Java client](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/clients/java/returning.java):

```java
import java.sql.*;
import java.util.*;

class returning {
    public static void main( String argv[] ) throws Exception {
        Class.forName( "org.postgresql.Driver" );
        String connectionURL = "jdbc:postgresql://localhost/testdb";
        Properties connectionProperties = new Properties();
        connectionProperties.put( "user", "luca" );
        connectionProperties.put( "password", "xyz" );
        Connection conn = DriverManager.getConnection( connectionURL, connectionProperties );

        String query = "INSERT INTO foo( rv ) "
            + " SELECT random() "
            + " FROM generate_series( 1, 10 ) "
            + " RETURNING pk, rv;";

        Statement statement = conn.createStatement();
        ResultSet resultSet = statement.executeQuery( query );
        while ( resultSet.next() )
            System.out.println( String.format( "The statement inserted pk = %d and a random value rv = %f ",
                                               resultSet.getLong( "pk" ),
                                               resultSet.getFloat( "rv" ) ) );

        resultSet.close();
        statement.close();
    }
}
```

that produces the very same result.
