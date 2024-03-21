---
layout: post
title:  "An Image-Pixelizer in Python (with ImageMagick)"
author: Luca Ferrari
tags:
- python
permalink: /:year/:month/:day/:title.html
---
A simple program to pixelize an image.

# An Image-Pixelizer in Python (with ImageMagick)

A few days ago a friend of mine challenged me with the need for an *image pixelizer*: given an image, the program has to divide it in nine squares, blurring all but the center one.

<br/>
**Image Magick to the rescue!**
<br/>
This has been my very first thought about it.

So, I quickly wrote a raw Python script that launched in turn two Image Magick applications: `convert` and `montage`. The former was used to produce nine images with the parts of the original image, the latter to reassemble them after having blurred every one except one.

Before continuing, having a source image as the following one

<center>
<br/>
<img src="/images/post/pixelizer/src.png" alt="original image" width="50%" />
<br/>
</center>

must produce something like the following one

<center>
<br/>
<img src="/images/post/pixelizer/dst.png" alt="final image" width="50%" />
<br/>
</center>


The first implementation of the script was working fine, but I was curious to implement it in a more "programmatic" way.


## Pure Python Implementation

This is how I implemented in pure Python ([the code is available on my GitHub repository](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/python/image_shuffler/img_pixelizer.py){:target="_blank"}):

<br/>
<br/>
```python
#!python

import sys
import itertools
import glob
import shlex
import subprocess
from wand.image import Image

# Example of invocation: img_pixelier.py src9.png

if __name__ == '__main__':
    src_file = sys.argv[ 1 ]


    src_image  = Image( filename = src_file )
    src_width  = src_image.width
    src_height = src_image.height

    pieces      = 3
    crop_width  = int( src_width / pieces )
    crop_height = int( src_height / pieces )

    dst_image = Image( width = src_width, height = src_height )
    for r in range(0, pieces):
        for c in range(0, pieces):
            start_at_x = c * crop_width
            start_at_y = r * crop_height
            segment = Image( filename = src_file )
            segment.crop( start_at_x, start_at_y, width = crop_width, height = crop_height )
            if c != r or c != int( pieces / 2 ) or r != int( pieces / 2 ):
                segment.blur( sigma = 20 )


            dst_image.composite( image = segment, left = start_at_x, top = start_at_y )

	dst_image.save( filename = 'blurred.png' )
```
<br/>
<br/>


In order to work, there is the need for the `wand` package, that is the binding to Image Magick.
The first step is to create an image from the given filename, and to compute the original dimensions of the image.
Then I assume to split the image into `pieces` blocks per row and colum, so that `pieces x pieces` parts will be produced, and hence I compute how large every single sub-image is going to be large.

Then I create a `dst_image` new image of the size of the original image, and loop over rows and columns performing a `crop` of the original image with adjusted offset depending on the piece I want to crop.

If the current sub-image is not the center one, i.e., `r` and `c` are different or not the half of their values, the sub image is also blurred.

At the end of the loop, I save the image on disk so that the final result is produced.


# Conclusions

Python is not my favourite language, but the `wand` binding for ImageMagick is really simple to use and to produce quickly good results.
