---
layout: post
title:  "ImageMagick, convert and 'operation not allowed by the security policy `PDF''"
author: Luca Ferrari
tags:
- linux
- imagemagik
permalink: /:year/:month/:day/:title.html
---
Does ImageMagick include a security policy?

# ImageMagick, convert and 'operation not allowed by the security policy `PDF''

Today a friend of mine asked to merge a few images into a single PDF document.
<br/>
How hard can it be? **`imagemagick` to a rescue**!
<br/>
Having a clean operating system, I installed it from scratch, via the package manager.
<br/>
However, when I tried to execte `convert` I got a strange and new (to me) error:


```shell
% convert 'mt 1.jpg'  'mt 2.jpg'  'mt 3.jpg' mt.pdf
convert-im6.q16: attempt to perform an operation not allowed by the security policy `PDF' @ error/constitute.c/IsCoderAuthorized/408.
```

What the hell?
<br/>
It turned out that ImageMagick now has a policy file located at `/etc/imagemagick-6/policy.xml` (or a different path), and that file includes a line for every type of conversion. For example, for PDF there is:

```shell
  <policy domain="coder" rights="none" pattern="PDF" />
```

that needs to be changed into something that allows the reading and/or writing privileges to the PDF *coder*:

```shell
  <policy domain="coder" rights="reader | write" pattern="PDF" />
```
