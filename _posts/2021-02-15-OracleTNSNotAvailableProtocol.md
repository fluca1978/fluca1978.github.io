---
layout: post
title:  "ORA-12516: TNS:listener could not find available handler with matching protocol"
author: Luca Ferrari
tags:
- oracle
- postgresql
permalink: /:year/:month/:day/:title.html
---
Not a very clear error message in my opinion...

# ORA-12516: TNS:listener could not find available handler with matching protocol

Today my clients started to give me back this error: `ORA-12516: TNS:listener could not find available handler with matching protocol`.
<br/>
This is an example of poor error message, in my opinion, and the reason behind the appearing of the above error was, in my case, the *xceeding of the available connections*
<br/>
And why do I believe this an example of poor error message? Well, because it does not indicate what the actual problem it is.
For example, PostgreSQL provides a clearer message error for a similar situation: `psql: error: could not connect to server: FATAL:  too many connections for role "luca"`.
