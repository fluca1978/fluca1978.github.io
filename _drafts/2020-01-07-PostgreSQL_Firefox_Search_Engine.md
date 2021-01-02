---
layout: post
title:  "Firefox and PostgreSQL Documentation Search"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to search directly within the PostgreSQL documentation from your Firefox web browser.

Firefox and PostgreSQL Documentation Search
---
The Firefox web browser supports several search engines, extensions by means of which you can insert a search string and get it passed to a specific site for search.
<br/>
It is possible to customize Firefox to search for a particular string within the PostgreSQL official documentation: the idea is to instrument the web browser to redirect the searching for thru the PostgreSQL web site via a `GET` URL.
<br/>
In order to achieve this, you need to install a customizable search engine, and then configure the shortcuts for enabling the web engine access.



## Custom Search Engine Setup

The first step consists of installing the [Custom Search Engine](https://addons.mozilla.org/en-US/firefox/addon/add-custom-search-engine/){:target="_blank"} to your Firefox web browser.
<br/>
Then, clicking on the main Firefox menu (the hamburger icon), select the *Add-ons* entry and then go to the *extensions* menu: you should see the new searching engine there. Check the engine is active and then click on the three dots button and select *Preferences*:

<br/>
<br/>
<center>
<img src="/images/posts/firefox/postgresql_search_engine_1.png" />
</center>
<br/>
<br/>

In the opened screen, edit a line to add the following details:
- *key*, I use `pg` as the default prefix to indicate I'm going to specify a PostgreSQL documentation search;
- *Search Engine Name*, set to `PostgreSQL` or any name it makes sense to you;
- *URL*, you have to set it to `https://www.postgresql.org/search/?q={searchTerms}`, where `{searchTerms}` is going to be replaced by firefox with the searching keywords;
- *Description*, whatever it makes sense to you, for example `PostgreSQL Official Documentation`.

<br/>
As you can imagine, the important parts are the *key* and the *URL*. Note that you can also add specific PostgreSQL versions by changing the URL to include a version number, do a few searches on the official web site and inspect the URL for other arguments.
<br/>
Once you have done, click on the button *Save Preferences* and then close the tab.

<br/>
<br/>
<center>
<img src="/images/posts/firefox/postgresql_search_engine_2.png" />
</center>
<br/>
<br/>


## Searching into the documentation

With the engine in place, you can search within the PostgreSQL documentation by means of inserting:
- `ms` to activate the custom search engine;
- `pg` to activate the PostgreSQL documentation search engine (this is the *key* specified above);
- any keyword you need to search into the documentation.
<br/>
As an example, imagine we want to search for the `CREATE INDEX` statement documentation; we need to enter:

<br/>
```
ms pg create index
```
<br/>

<br/>
<br/>
<center>
<img src="/images/posts/firefox/postgresql_search_engine_3.png" />
</center>
<br/>
<br/>

and pressing `enter` the search will go thru the PostgreSQL documentation web site:


<br/>
<br/>
<center>
<img src="/images/posts/firefox/postgresql_search_engine_4.png" />
</center>
<br/>
<br/>


# Conclusions

I personally don't like very much the way Firefox allows for a search customization: having to type a shortcut to activate the search engine and another one to specialize the search engine seems to me too much work.
However, it can result useful when you live in Firefox and want to quickly search for a PostgreSQL tip!
