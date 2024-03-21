---
layout: post
title:  "pgagroal-cli minor bug fixes"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
A few changes to a part of `pgagroal`.

# pgagroal-cli minor bug fixes


In the past days I pushed a few troophy patches to `pgagroal-cli`, the command line tool to administer a `pgagroal` connection pooler, in order to fix minor issues that produced unattended results.

*The bug were all harmless*, since they only affected what the `pgagroal-cli` was producing as output to the user, but could have been confusing for some use cases, hence the need to fix them.

There is quite a momentum around the `pgagroal` project, and the activity around the issues has increased with the arrival of new contributors!
