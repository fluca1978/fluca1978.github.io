---
layout: post
title:  About API coherence
author: Luca Ferrari
categories: api, java, c++, perl, development
permalink: /:year/:month/:day/:title.html
---
Is the *getter/setter* API coherent across your library?

## About API Coherence
-----
I've already spent some words on the [setters and getters](https://www.google.it/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwiM2Jmzhv_UAhXFXhQKHTtBAfgQFggoMAE&url=http%253A%252F%252Ffluca1978.blogspot.com%252F2009%252F06%252Fsetter-getter-o-entrambi.html&usg=AFQjCNGHcLXyCXw_9PJ65aoJnOs9E88oaQ) subject, and I have to admit I hate with a  passion the way Java Beans implements them.
There are several ways I don't like the Java Beans specification about setters and getters, most notably the already discussed fact that it does not allow for a *fluent* interface, that is an handy way to concatenate setter calls. The reason for this is that Java requires a *setter* to return nothing (i.e., `void`), so that you have to use *setters* as follows:

    foo.setWidth( 30 );
    foo.setHeight( 50 );

while if the `setWidth` and `setHeight` does allow for a return type, in particular the same object they are mutating, the above code becomes a lot more compact and less error prone:

    foo.setWidth( 30 );
       .setHeight( 50 );

Why is the above less error prone? Because I'm mutating the *very same object*, no way I can mess with variables with similar names or something wrongly suggested by my IDE.
Well, to be honest, the fluent interface requires an extra slot on the stack to place the object to return, so chances are it is not a popular approach to save some memory access, but I think developers would have a strong benefit for change.

The other fact that makes me hate the Java Beans *s|g-etter* notation is the forced adoption of placing a prefix in front of each method.
Do you a property named `width`? You will end up with a couple of methods called `getWidth()` and `setWidth(..)`. There is no practical reason for having such suffixes, since I can only say a method from a variable due to the presence of brackets. In other words `width` is different from `width()` (and no, spaces are not a problem!).
This approach also lead to a few inconsistencies around the Java library itself, and this is due probably to different ages and evolutions of the API itself. But before going into deep here, let's review the naming convetion another time.
In Java, if you have a property `baz` you could have the following accessors:

-   `getBaz()` to read;
-   `setBaz(..)` to write.

What about a method that perform some action on `baz`, as a `normalizeBaz()`? Well, it is quite clear that *non-accessor methods never have a `set` or `get` suffix*. This rule of thumb could make the specification a little more easy to tolerate, as it allows to quickly say a computational method from an accessor one. The problem, in my opinion, is that Java engineers fears developers to write stupid dummy code and force them to avoid it (yeah, just think about the lack of operator overloading).
Again, my personal answer is: "who cares?".
Allow me to explain: I often end up with code that looks like the following


    public final void setWidth( int w ){
     if ( w < 0 )
        w = 0;
      width = w;
    }

    public final int getWidth(){
      if ( width < 0 )
         width = 0;
      return width;


The above code is <span class="underline">wrong</span> (in a pure Java Beans sense): accessor perform some kind of sanity on possible tainted data. Therefore my accessors also <span class="underline">do</span> computation, and therefore I cannot say a computational method from an accessor one without inspecting the code (or reading the documentation, that we all write, right?).

Assuming therefore that is a day-by-day needing for some accessors to perform some computation, let's go back to the naming coherency.
Is a Java `String` a bean? Apparently no, because its length is obtained via a read only accessor named `length()`.
Is a collection a bean? Apparently no, because its size is obtained via a read only accessor named `size()`.
Now, I hear you screaming "of course, they are not supposed to be writable properties!", and I agree with you, but then why they should not adhere to `get` prefix specification? And please note they should not be computational method, since the sizes are pretty well known at the time you ask for them.

Ok, let's see someone else that does things in a more coherent way. Let's examine the [Qt library](http://doc.qt.io/qt-5/reference-overview.html), which in my opinion is a very robust and great piece of code, a library we all should learn from.
The approach used there is quite simple: if you have the property `baz` you end up with

-   `baz()` to read;
-   `setBaz()` to write.

Note that the read accessor is perfectly coherent with any other version of the API, and could be coherent even with other Java classes.
Do you need a computational method? Just named it after the property, such as `computeBaz()`.
Clear, simple, coherent. No need to uppercase first the property in every accessor.

As for Java, even Qt does not implement the fluent interface, and I suspect the reasons are the same: avoiding an extra return slot on the stack.

Now, a syntactic sugar of the Java Beans specification is that *all* the getters and setters are grouped alphabetically, and this could allow a developer for a faster lookup once the IDE sort members and methods alphabetically. On the other hand, this is not a need for a C++ library, since usually variables and methods are kept in separate source files. This is a really small advantages compared to all the extra typing of `get` in front of reader accessors, and the inconsistencies you can find across the library.
