---
layout: post
title:  "Mojolicious: prefilling an email field with a Perl expression"
author: Luca Ferrari
tags:
- mojolicious
- perl
permalink: /:year/:month/:day/:title.html
---
A little trap when dealing with Mojolicious forms.

# Mojolicious: prefilling an email field with a Perl expression

I am acquiring knowledge with Mojolicious templates and forms, that are obtained via `Mojolicious::Plugin::TagHelpers`.

The problem I was facing, that I've reported (and solved by myself) in [this forum thread](https://github.com/mojolicious/mojo/discussions/2159){:target="_blank"}, was about pre-filling an email input field with data coming from the underlying model.

My first attempt was:

<br/>
<br/>
```perl
%= email_field 'email' => <%= $user && $user->is_logged_in ? $user->email : '' %>,  id => 'email', class => 'form-control'
```
<br/>
<br/>

where, clearly, `$user` is the model.

This approach was wrong, since it was producing an HTML like the following:

<br/>
<br/>
```html
 <input name="email" type="email" value="foo@bar.com">,  id => 'email', class => 'form-control' %>
```
<br/>
<br/>

that clearly was poorly formatted with all the extra attributes outside of the `input` HTML tag.


The idea came to me after reasoning about the starting tag `%=`: **this is a Perl expression line starter**, and therefore everything that follows is managed as a Perl expression, as for instance the fat comma.
Therefore, the right solution is to let the expressio to parse a Perl piece of code:

<br/>
<br/>
```perl
%= email_field 'email' => ( $user && $user->is_logged_in ? $user->email : '' ),  id => 'email', class => 'form-control'
```
<br/>
<br/>

that correctly produces the HTML that follows:

<br/>
<br/>
```html
<input class="form-control" id="email" name="email" type="email" value="foo@bar.com">
```
<br/>
<br/>
