---
layout: post
title:  "Test::More : diag versus note"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
A trivial, but often abused, difference between two methods in the `Test::More` suite.

# Test::More : diag versus note

I tend to add a lot of *self-explainatory messages* within my single tests, so that once I run a single test I can see what is happening behind the hood and have a chance to understand (quicker) why a test is failing.

There are two main methods to this purpose:
- **`note`**
- **`diag`**

Both seems to work the same, outputting to `STDERR`, the given message, so play the same role as `say` in command line application.

However, while `note` is silenced when running in an harness (e.g., `prove`), the `diag` output is always shown to the user. Therefore, the latter method should be used only for really important messages and not as a debug or internal echoing of stuff.
