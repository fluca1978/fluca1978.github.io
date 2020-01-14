---
layout: post
title:  "Perl5 -> Perl6: sprintf"
author: Luca Ferrari
tags:
- raku
- perl
- programming
permalink: /:year/:month/:day/:title.html
---
A simple snippet about some idiom I carry on from my Perl 5 day-to-day work

## Perl5 -> Perl6: sprintf
-----

I love ```sprintf```. After the **wow!** string concatenation hype, I found myself a lot more comfortable in adopting ```sprintf``` and alike for string formatting. It looks a lot nicer and more readable to me.
Of course, it has drawbacks, as a lot of *variadic* methods: you can miss a parameter or fail its type. However, once set up, it is a lot
stable than string concatenation (and can be translated in an easier way).

The above means that a lot of messaging in my Perl programs use ```sprintf``` to, and often I use the following piece of code in Perl 5:

``` perl
 say sprintf "Inizio elaborazione modo %s (%s) su file di output [%s] %s limite",
            $current_mode,
           %available_modes{ $current_mode },
           $output,
           ( $limit.defined ? sprintf '%d righe', $limit  : 'SENZA' ) );
```

Despite the fact that the string is in italian, you can see how simple it is.

In Perl 6 I can enhance the above using the **object-oriented** nature and the fact that ```sprintf``` is now a method of ```Str```:

``` perl
 say "Inizio elaborazione modo %s (%s) su file di output [%s] %s limite"\
 .sprintf( $current_mode,
           %available_modes{ $current_mode },
           $output,
           ( $limit.defined ?? '%d righe'.sprintf( $limit ) !! 'SENZA' ) );
```

I like this form the most because it is a little more *imperative*: you order to **say** something that is then **formatted**, and not to **say-format** something.

Please note the use of the ```unquote``` operator to keep things on different lines. It is not mandatory in this case, but it can be a good habit
for when Perl 6 requires it. Also note the different ternary operator based on ```?? !!```.
