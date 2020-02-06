---
layout: post
title:  "PHP Namespaces Oddities"
author: Luca Ferrari
tags:
- php
- programming
permalink: /:year/:month/:day/:title.html
---
It seems to me PHP cannot get right even namespaces!

# PHP Namespaces Oddities

*It's time for flame!*
<br/>
**I have a strange relationship with PHP: I hate it even if it has served me well and still pays my bills!**
<br/>
<br/>
I have to admit that I'm not a PHP purist, and my applications tend to be quite simple and modular. Since a couple of years I progressively started using *namespaces*.
<br/>
After all, how hard can it be?
<bR/>
I've used namespaces in pretty much every language, ranging from C++ to my beloved Perl.
<br/>
<br/>
But the way PHP implements namespaces is ... ugly!
<br/>
Why?
<br/>
The first reason is that **when you are in a namespace everything outside the namespace must be fully qualified**. Let's see this in action:

```php
namespace fluca1978\database;

class Driver {
   public function connect() {
      // get an object outside of the namespace
      $query = new \Query;
   }
}
```

**Why do I have to call `\Query` instead of `Query`**? Because once I'm in the namespace, any reference is supposed to be within the very same namespace. 
<br/>
**This produces a lot of runtime errors in my applications because I simply forget somewhere to `\` *global* names**, and that is bad to me!
<br/>
<br/>
Reason number two: **when you need to use a namespace, you have to repeat the trailing part.**
<br/>
Let's see this in action:

```php
use namespace fluca1978\database;

...

$driver = new \database\Driver;
```

why do I have to repeat the `\database` part of the namespace since I'm already `use`-ing it? Why is it not importing the new names into the local namespace?

# Conclusions

I'm sure PHP has strong reasons for implementing the namespace resolution in such a dirty way, and I could guess it has something to do with backward compatibility.
<br/>
Being used to other languages namespaces, I could say this is not what I expect to be from a `namespace` and `use` directives. **Clearly I'm not the PHP mood about namespaces!**

