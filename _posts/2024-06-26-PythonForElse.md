---
layout: post
title:  "Python for...else construct"
author: Luca Ferrari
tags:
- python
permalink: /:year/:month/:day/:title.html
---
Something interesting on a language I don't like.

# Python for...else construct

As [documented in the for loop syntax](https://docs.python.org/3/reference/compound_stmts.html#the-for-statement){:target="_blank"} the `for` loop allows an `else` block, and so does [the while loop](https://docs.python.org/3/reference/compound_stmts.html#the-while-statement){:target="_blank"}, so that you can write something like:

<br/>
<br/>
```python
r = range( 2 )
for i in r:
    print( f"Looping {i}" )
else:
    print( "Else block" )

r = None
while r is not None:
    print( f"While {r}" )
    r -= 1
    if r == 0:
        r = None
else:
    print( "While else block" )

```
<br/>
<br/>

The idea is quite interesting, after all, even if I don't like how it is implemented.

As stated in the documentation, the `else` block is executed **when the iterator is exhausted**, hence when the looping ends (or it never starts).
Therefore, in the first `for` example, the `else` block is executed once the iteration ends, and in the `while` it is immediatly executed since the iterator is empty (and an empty iterator is seens as an exhausted one).


The usage of an `else` block simplifies the code because you don't need anymore an `if` to check if your iterator is empty or not. The problem is that the `else` block is executed even if the iterator was full!

I was expecting something like:

<br/>
<br/>
```python
r = range( 2 )

if r is not None:
	for i in r:
		...
else:
	print( "Iterator empty" )
```
<br/>
<br/>


while effectively it is implemented as it was written as:

<br/>
<br/>
```python
r = range( 2 )
done = False

for i in r:
    ...
	done = True



if not done or r is None:
	print( "Iterator is empty" )
```
<br/>
<br/>


that makes much less value according to me.
