---
layout: post
title:  "dbicdump: using PostgreSQL schemas as package separator in produced Perl classes"
author: Luca Ferrari
tags:
- perl
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A way to instrument `dbicdump` to use PostgreSQL schemas as package separators.

# dbicdump: using PostgreSQL schemas as package separator in produced Perl classes

Perl `DBIx::Class` is a great Object Relational Mapper (ORM), and I use it regularly with `dbicdump`, which is a tool to **synchronize** your existing database structure with the classes your program is going to use.

PostgreSQL being PostgreSQL, a great rock solid database we all love, allows us to organize tables into *schemas*, a flat namespace that is usually transparent to the user because the default schema, `public`, is always into the `search_path` for every user.

But how to take advantage of PostgreSQL schemas and `DBIx::Class` packages?

Well, it turned out that this is possible, with a little customization of the way you sycnhronize your own data structure.


## Example Database

Assume we have an example database with a couple of tables, namely `products` and `orders`, each one replicated into two different schemas named respectively `italy` and `japan`. Note, this is probably not the better design for your database, but it does serve only as an example to get a quick and easy idea of how to achieve things.

The database results as follows:

<br/>
<br/>
```sql
dbic=> CREATE SCHEMA italy;
CREATE SCHEMA
dbic=> CREATE SCHEMA japan;
CREATE SCHEMA
                                           ^
dbic=> CREATE TABLE italy.product( pk serial,
            code text,
	        description text,
			primary key( pk ),
			unique( code ) );
CREATE TABLE

dbic=> CREATE TABLE japan.product( pk serial,
          code text,
		  description text,
		  primary key( pk ), unique( code ) );
CREATE TABLE

dbic=> CREATE TABLE italy.orders( pk serial,
				  product int not null,
				  qty int default 0
				  , primary key ( pk )
				  , foreign key( product ) references italy.product( pk ) );
CREATE TABLE

dbic=> CREATE TABLE japan.orders( pk serial,
				product int not null,
				qty int default 0
				, primary key ( pk )
				, foreign key( product ) references japan.product( pk ) );
CREATE TABLE

```
<br/>
<br/>


Let's populate the `products` table with a few rows:

<br/>
<br/>
```sql
dbic=> insert into italy.product( code, description )
       values( 'it01', 'An italian product' );
INSERT 0 1

dbic=> insert into japan.product( code, description )
       values( 'jp01', 'A japanese product' );
INSERT 0 1

dbic=> insert into japan.product( code, description )
       values( 'jp02', 'A japanese product' );
INSERT 0 1


dbic=> insert into italy.orders( product, qty )
		    select p.pk, ( random() * 100 )::int
		    from italy.product p, generate_series( 1, 5 ) v;
INSERT 0 5


dbic=> insert into japan.orders( product, qty )
			select p.pk, ( random() * 100 )::int
			from japan.product p, generate_series( 1, 5 ) v;
INSERT 0 10


```
<br/>
<br/>


## Dumping the schema via `dbicdump`

In order to dump the schema via `dbicdump`, you need to pass several additional options:
- the schema names to dump, in our example `italy` and `jpana`;
- the **moniker parts to use**, that is how the class name will be built. By default the `moniker` is set to `name`, that means it will call the `name` method (i.e., the table name). In our example, we need to use both `name` and `schema`, with the latter before the former;
- set the **moniker parts separator**, that is the character to use to separate the parts of the name. since we want to produce modules with their namespace, we will use the Perl namespace separator, that means `::`.

This translates to a command line like the following:

<br/>
<br/>
```shell
% dbicdump -o dump_directory=/home/luca/tmp            \
           -o components='["InflateColumn::DateTime"]' \
		   -o moniker_parts='["schema", "name"]'       \
		   -o moniker_part_separator='::'              \
		   -o db_schema='["public", "italy", "japan"]' \
		   Example::Schema                             \
		   'dbi:Pg:dbname=dbic;host=rachel;port=5432'  \
		   luca superSecretPassword

Dumping manual schema for Example::Schema to directory /home/luca/tmp ...
Schema dump completed.

```
<br/>
<br/>

The parameters passed to `dbicdump` are the followings:
- `dump_directory` where to store the Perl code produced;
- `components='["InflateColumn::DateTime"]'` this is not mandatory for this post example, but is a good habit to get automatic date/time data type conversions;
- **`moniker_parts='["schema", "name"]'`** this tells `dbicdump` to compose the name of a class mapped onto a table as the schema name plus the table name, which is what we want;
- **`moniker_part_separator='::'`** this tells `dbicdump` to use the Perl name separator (i.e., package separator `::`) between the schema name and the table name;
- **`db_schema='["public", "italy", "japan"]'`** this tells `dbicdump` to dump the `public`, `italy` and `japan` schemas, i.e., where to look for tables.

The resulting tree is as follows:

<br/>
<br/>
```shell
% tree Example
Example
├── Schema
│   └── Result
│       ├── Italy
│       │   ├── Order.pm
│       │   └── Product.pm
│       └── Japan
│           ├── Order.pm
│           └── Product.pm
└── Schema.pm

```
<br/>
<br/>

That is the table `italy.products` has been translated to `Italy::Product`, and the other similarly.


## Using the table structure

In order to use the Perl classes, and most notably, to query the tables, there is the need to pass the class names into the `resultset` method.

As an example:

<br/>
<br/>
```perl
#!perl

use v5.40;
use Example::Schema;
use Example::Schema::Result::Italy::Product;
use Example::Schema::Result::Japan::Product;


my $db = Example::Schema->connect(  'dbi:Pg:dbname=dbic;host=rachel;port=5432' ,
				    'luca',
				    'superSecretPassword' );


my @italian_products  = $db->resultset( 'Italy::Product' )->all;
my @japanese_products = $db->resultset( 'Japan::Product' )->all;

say "There are " . scalar( @italian_products ) . " italian products";
say "There are " . scalar( @japanese_products ) . " japanese products";


for my $product ( @italian_products ) {
    say "[ITALY] " . join( " | ", $product->code, $product->description );
}

for my $product ( @japanese_products ) {
    say "[JAPAN] " . join( " | ", $product->code, $product->description );
}

```
<br/>
<br/>


Note how the `resultset` method does not accept the table name, rather the Perl module name.
In other words, `italy.product` does not work, while `Italy::Product` works.

In fact, enabling `DBIC_TRACE` and running the sample program produces the following output:

<br/>
<br/>
```shell
% export DBIC_TRACE=1

% perl test.pl
SELECT me.pk, me.code, me.description FROM italy.product me:
SELECT me.pk, me.code, me.description FROM japan.product me:
There are 1 italian products
There are 2 japanese products
[ITALY] it01 | An italian product
[JAPAN] jp01 | A japanese product
[JAPAN] jp02 | A japanese product

```
<br/>
<br/>

As you can see, the queries are correctly translated into `<schema>.<tablename>`. This is thanks to the fact that the `table` method in every class has been invoked with the fully qualified name. As an example:

<br/>
<br/>
```shell
% less Example/Schema/Result/Italy/Product.pm

...
__PACKAGE__->table("italy.product");
...
```
<br/>
<br/>


## Using Relationships

Once it is clear how the tables are named, it is quite simple to query relationships.
Let's do it programmatically first:

<br/>
<br/>
```perl
#!perl

use v5.40;
use Example::Schema;
use Example::Schema::Result::Italy::Product;
use Example::Schema::Result::Japan::Product;


my $db = Example::Schema->connect(  'dbi:Pg:dbname=dbic;host=rachel;port=5432' ,
				    'luca',
				    'luca' );

my @italian_orders  = $db->resultset( 'Italy::Order' )->all;
my @japanese_orders = $db->resultset( 'Japan::Order' )->all;

for ( @italian_orders ) {
    say sprintf "[ITALY] qty = %d for product %s" ,
	    $_->qty,
	    join( "|", $_->product->code, $_->product->description );

}

for ( @japanese_orders ) {
    say sprintf "[JAPAN] qty = %d for product %s" ,
	    $_->qty,
	    join( "|", $_->product->code, $_->product->description );

}

```
<br/>
<br/>



But let's assume we want to query all the products that have at least one order of a given quantity (again, this is an example).
This can be done as follows:

<br/>
<br/>
```perl
my @italian_products = $db->resultset( 'Italy::Order' )
	->search_related( 'product' )
	->search( { qty => 36 } )
	->all
	;
```
<br/>
<br/>

Let's dissect this:
- `resultset( 'Italy::Order' )` is what we search first;
- `search_related( 'product' )` is what we join and extract then;
- `search( { qty => 36 } )` is the search condition (i.e., the `WHERE` clause);
- `all` is the materialization of the result set.

The above translates to the following query (again `DBIC_TRACE` to get information about):

<br/>
<br/>
```sql
SELECT product.pk, product.code, product.description
FROM italy.orders me
JOIN italy.product product
ON product.pk = me.product WHERE ( qty = ? ): '36'
```
<br/>
<br/>



Wait a minute! What is that `product` name that appears into the `search_related` method? Why is not `Italy::Product` as before?
This is due to how `DBIx::Class` handles the relationships: every relationship gets a name that is used to tell DBIx what to join.
Inspecting `Italy::Order` you can find something as follows:

<br/>
<br/>
```perl
__PACKAGE__->belongs_to(
  "product",
  "Example::Schema::Result::Italy::Product",
  { pk => "product" },
  { is_deferrable => 0, on_delete => "NO ACTION", on_update => "NO ACTION" },
);

```
<br/>
<br/>

The string `"product"` is the name of this join relationship, that has to be used when telling DBIx to join another table from `Italy::Order`.

This is a kind of trick used by DBIx, so that having an `Order` you can simply spell `$order->product->code` and it will work fine. You can rename such association as you like (having care of not irritating `dbicdump` self generated code), and use the name you like the most in joining, but I strongly recommend you to avoid this. Rather, design better your tables.


# Conclusion

`DBIx::Class` is a very powerful and elegant ORM, and `dbicdump` allows you to organize your code in packages following the same clean order you can achieve with PostgreSQL schemas.
