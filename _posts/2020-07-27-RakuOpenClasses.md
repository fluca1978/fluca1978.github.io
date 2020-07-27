---
layout: post
title:  "Raku vs Ruby Open Classes and Monkeys"
author: Luca Ferrari
tags:
- raku
- ruby
permalink: /:year/:month/:day/:title.html
---
Do you know that Raku supports open classes?

# Raku vs Ruby Open Classes and Monkeys

Ruby is famous, among other interesting features, for its **[open classes](https://ruby-lang.co/ruby-open-class/){:target="_blank"}**. The idea is that you can inject *behaviors** into existing classes without having to inherit, neither decorate, neither re-define them.
<br/>
<br/>
**I'm not going to discuss if this features is useful or Object Oriented related**, this is a discussion I'm not interested in. I think there are times when such capability is useful, and quite frankly my [thesis project BlackCat](/projects){:target="_blank"} and its successor `[WhiteCat](https://github.com/fluca1978/WhiteCat){target="_blank"}` both aimed at doing something really nasty that could had been a piece of cake with *open classes*.
I also believe that, under the hood, it is a very good idea to have open classes, and for example implementing a run-time environment can take a clear advantage of this concept. **On the other hand, I don't think open classes are for mere mortals and everyday usage**.
<br/>
<br/>
However, open classes is still something really important to Ruby-ers, and is one of the motivations sometimes are told to me to explain the love for such language.
<br/>
<br/>
And Raku has open classes too!
<br/>
<br/>
Well, they are not called *open classes*, but you can mimic the very same Ruby behavior with `[augment](https://docs.raku.org/syntax/augment){:target="_blank"}`.
<br/>
<br/>
Let's see first a Ruby example:
<br/>
<br/>
```ruby
#!ruby

class String
  def PostgreSQL
    "PostgreSQL is an amazing database!"
  end
end


puts "Oracle is my favourite database!".PostgreSQL
```

<br/>
<br/>
The above piece of code enhances the `String` class, that is an internal class of Ruby core, with the `PostgreSQL` method. This means, that the `puts` line does not print the message you are expecting, rather what the method `PostgreSQL` provides:

<br/>
<br/>
```shell
% ruby oc.rb 
PostgreSQL is an amazing database!
```
<br/>
<br/>

Let's now see how this can be achieved in Raku:

<br/>
<br/>
```perl6
#!raku

use MONKEY-TYPING;
augment class Str {
    method PostgreSQL() { "PostgreSQL is an amazing database!" }
}

say "Oracle is my favourite database!".PostgreSQL;

```

<br/>
<br/>

The example is pretty much the same as its Ruby counterpart, except that it refers to the `Str` class that is the Raku version of the Ruby `String`. The method `PostgreSQL` is added similarly.
The trick here is that you need to inform Raku that you are going to do something *potentially dangerous* as extending an existing class, and you inform Raku about your intentions with the `augment` special keyword.
However, that does not suffice: you need to double assure Raku you know what you are doing by using the `MONKEY-TYPING` pragma.
<br/>
The final result is the same as for the Ruby implementation:


<br/>
<br/>
```shell
% raku oc.p6
PostgreSQL is an amazing database!
```
