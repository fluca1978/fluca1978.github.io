---
layout: post
title:  "krunner and PostgreSQL Documentation Search"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How to search directly into the PostgreSQL documentation from your Plasma desktop.

krunner and PostgreSQL Documentation Search
---
If you, like me, are addicted to Plasma, the KDE desktop, you probably already know about *krunner*, an **application launcher on steroids**.
<br/>
`krunner` allows you to quickly launch, kill, switch to and manage applications, as well as executed computations and, most notably **web searches**. In fact, krunner exploits the *Konqueror* shortcuts for web searches. Konqueror is the default web browser for KDE/Plasma (since KDE version 2), and allows for a quick customization of shortcut that enable you to redirect a string thru a search engine.
As an example, by default Konqueror has the `dd` and the `gg` shortcuts: the former enbles the search of the remaining part of the string thru *DuckDuckGo*, while the latter thru *Google*.
<br/>
So, what does it take to get krunner integrated with the PostgreSQL official documentation search engine?
<br/>
There is no much work to do, after all, and in fact it does suffice to:
1) create a new Konqueror shortcut;
2) no, there are no other steps involved!
<br/>
The good news is that you can configure whatever you want by the krunner interface itself.

## Configure krunner
First of all, launch krunner by hitting `ALT + F2` or `ALT + <space>`, then click on the setup icon on the left of the bar

<br/>
<br/>
<center>
<img src="/images/posts/plasma/postgresql_search_engine_1.png" />
</center>
<br/>
<br/>

In the dialog window, scroll to the *Web Shortcuts* line and click on the *configure* icon.

<br/>
<br/>
<center>
<img src="/images/posts/plasma/postgresql_search_engine_2.png" />
</center>
<br/>
<br/>

In the opened dialog, after having searched for the *key* sequence you want to insert, click on the *New* button to create a new shortcut.

<br/>
<br/>
<center>
<img src="/images/posts/plasma/postgresql_search_engine_3.png" />
</center>
<br/>
<br/>



Fill the dialog as you find appropriate, but with regard to the `Shortcut URL` place `https://www.postgresql.org/search/?q=` and then hit the button on the right to insert the query parameters (`\{@}'), so that the ending result is `https://www.postgresql.org/search/?q=\{@}`.
<br/>
Place a shortcut in the `Shortcuts` entry, separaed by comma, for example `pg`, then `postgres` and last `postgresql`, so that you will be able to inject a search by a short or common character sequence.


<br/>
<br/>
<center>
<img src="/images/posts/plasma/postgresql_search_engine_5.png" />
</center>
<br/>
<br/>

Apply the changes and get ready for your PostgreSQL related queries.

## Searching for something PostgreSQL-related

It is now time to test the searching shortcut:
- launch krunner by hitting `ALT + F2` or `ALT + <space>`;
- enter `pg:` to activate the search engine
- insert a PostgreSQL string and press `<enter>`.

<br/>
<br/>
<center>
<img src="/images/posts/plasma/postgresql_search_engine_6.png" />
</center>
<br/>
<br/>


and the result will popup in your default web browser (*that is not mandatory to be Konqeuror!*).


<br/>
<br/>
<center>
<img src="/images/posts/plasma/postgresql_search_engine_7.png" />
</center>
<br/>
<br/>


# Konqueror and Web Shortcuts

As already written, `krunner` exploits the Konqueror Web Shortcuts, and in fact [I wrote an article (italian)](https://fluca1978.github.io/2008/01/28/ricerca-diretta-nella-documentazione-di.html){:target="_blank"} back in *2008* about the configuration of Konqueror to access the PostgreSQL documentation. I also asked for that article to appear on the ITPUG official web site, without any success, but this is another story.


# Conclusions

`krunner` is an amazing piece of software, that I totally use every day and every moment to the extent that I do not more use a lot of icons to start applications and tasks, but simply pass a few characters to krunner and let it do the heavy work for me.
<br/>
Being able to integrate the PostgreSQL documentation search into krunner represent a huge adavantage for every PostgreSQL and Plasma user.

