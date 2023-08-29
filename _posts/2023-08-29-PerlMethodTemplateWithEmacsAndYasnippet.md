---
layout: post
title:  "Perl class method templating with YASnippet in Emacs"
author: Luca Ferrari
tags:
- emacs
- perl
permalink: /:year/:month/:day/:title.html
---
A minimal template for boring tasks like adding a method to a class.

# Perl class method templating with YASnippet in Emacs

Emacs is a great editor, we all know!
<br/>
Perl ia a great language, we all know!
<br/>

Using Emacs to edit Perl code is fine, but thanks to YASnippet it can be awesome!

<br/>

One task I often have to do is to add a method to a Perl class. It's something really simple, and I don't use (yet) method signatures, so it is really a short piece of code. But it is boring and repetitve, so it is a good candidate for a *template*!
<br/>
And of course, a piece of code must have documentation, so I want also to add some *POD* stuff to the method.

Therefore I created a simple template within my YASnippet configuration to aim at method creation.

## The method template: the workflow

The following is a short video about how easy it is to create a new method with arguments using the template:

<br/>
<br/>
<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/k7fRDUpyIoE?si=LYGxapayoeMu7Rsn" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
</center>
<br/>
<br/>

As you can see, all starts with the documentation at first. The template asks for the name of the method within the POD documentation section, and updates the name of the method in the code. Then it ask for the usual stuff, like the method description and the return value. If the method does not return a specific value, I leave the default sentence. Sometimes my methods throw an exception, and I want to clearly document that as well, so the template asks for it and I can leave the default sentence here too.
<br/>
Last, my method could accept other arguments other than `$self`, so the template places the cursor in the `my` parameter assignment line. Last, it throws the cursor at the method body, so the real fun, i.e., writing Perl code, can begin!


## The method template: the code

here it is the code for the method template, at least the one that corresponds to the above video. I may change it slightly in the future to be more efficient in my own workflow, but use it as you like (and please send a *thank you* or give some credits if you wish!):

<br/>
<br/>
```
# -*- mode: snippet -*-
# name: method ... { ... }
# key: method
# --


=head2 ${1:method_name}

${2:method_description}

Return value: ${3:no particular value is returned.}

Throws: ${4:no exception is thrown.}

=cut
sub $1 {
    my ( $self $5) = @_;

    $0
}

```
<br/>
<br/>

I have the above saved as `method` file under the `cperl-mode` subdirectory of my snippet directory.


# Conclusions

YASnippet is a very powerful template engine that I'm using more everyday, and that now I'm also heavily putting into my Perl workflow.
