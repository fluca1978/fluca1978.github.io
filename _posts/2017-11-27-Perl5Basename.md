---
layout: post
title:  "Perl5 and File::Basename as inline trick"
author: Luca Ferrari
tags:
- perl
- programming
permalink: /:year/:month/:day/:title.html
---
One of strenghts of Perl 5 is the huge amount of modules that do what you need. But sometime I got stuck with modules don't do what they want...

# Perl5 and File::Basename as inlike trick!
-----

Today I asked for help in fixing a Perl 5 program of mine.

The program was working really fine, so nothing really problematic was here, it was just that feeling that things could be done better, shorter, in a more Perl-way!
And the suggestion came out from the [Perl Beginners Mailing list](https://www.nntp.perl.org/group/perl.beginners/2017/11/msg126710.html), yeas sometimes you need to get to the root...

So, here's the diff of the code I committed today:


```perl
-  if ( $current_object->{ type } eq 'dir' ){
-      my @remote_dirs = File::Spec->splitdir( $remote_path );
-      pop @remote_dirs;
-      $remote_path    = File::Spec->catdir( @remote_dirs );
-  }
+  $remote_path = dirname( $remote_path ) if ( $current_object->{ type } eq 'dir' );
```

The thing was this: I needed to remove the very last directory from a complex path, something like `/a/b/c/d/f` so that only the absolute beginning part up to `/a/b/c/d/` remains. Since I'm used to `File::Specs`, and I cannot find there anything helping with my tiny problem, I split up the path into parts, throwing away the last element and reassembling the parts to a full path. As you can see, all the above is conditioned by an `if` statement, and that prompted my brain for a better way to do it with a postfix condition, that seems to me a better way to read this operation.
Anyway, in order to adopt a postfix conditional I need to tear down the block of code to a single command or a single *pipeline*, but I was not finding out anything useful in `File::Spec`.

And then `File::Basename` came as a rescue with its `dirname` method that *blindly returns the path up to the last to end part*, exactly what I needed for.

And yes, I could have done it via regular expressions, but I was looking for a more portable way to do it.

Again, a great lesson learned from being humble and adopting the great [CPAN](http://search.cpan.org).
