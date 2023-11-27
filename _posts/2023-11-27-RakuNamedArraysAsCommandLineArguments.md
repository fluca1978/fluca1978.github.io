---
layout: post
title:  "Passing named arrays as command line arguments in Raku"
author: Luca Ferrari
tags:
- raku
permalink: /:year/:month/:day/:title.html
---
A simple reminder on how to pass named argument arrays to a MAIN method.

# Passing named arrays as command line arguments in Raku

Raku provides a `MAIN` method that, when present, can define the arguments the application expects.
This is very useful, because it allows for argument checking and definition.

However, while passing simple positional arguments to a `MAIN` is simple, when dealing with named arrays you have to repeat the positional name for every single element.

As an example, assume we have written the following simple application:

<br/>
<br/>
```raku
sub MAIN( :@books, :@years )
{
    say "Luca Ferrari has written the following books:";
    for ( @books [Z] @years ) {
		say "- $_[0] ( $_[1] )";
    }
}

```
<br/>
<br/>

In order to define the `@books` and `@years` you have to specify the options `--books` and `--names` respectively on the command line.
And you have to repeat them every time you need to append a new element to the array.

**The beauty part is that you don't have to place all the arguments grouped, but you can mix arguments and Raku will deal them!**

In other words, the following two invocations are equivalent:

<br/>
<br/>
```shell
% raku books.p6 --books="Learn PostgreSQL" \
	 --books="Learn PostgreSQL" \
	 --books="PostgreSQL 11 Server Side Programming" \
	 --years=2023 \
	 --years=2020 \
--years=2018

% raku books.p6 --books="Learn PostgreSQL" \
	 --years=2023 \
	 --books="Learn PostgreSQL" \
	 --years=2020 \
	 --books="PostgreSQL 11 Server Side Programming" \
	 --years=2018

```
<br/>
<br/>


and in both cases the output will be:

<br/>
<br/>
```shell
Luca Ferrari has written the following books:
- Learn PostgreSQL ( 2023 )
- Learn PostgreSQL ( 2020 )
- PostgreSQL 11 Server Side Programming ( 2018 )


```
<br/>
<br/>
