---
layout: post
title:  "PostgreSQL Literate Programming with GNU Emacs"
author: Luca Ferrari
tags:
- emacs
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
GNU Emacs is great! I can prepare my slides with PostgreSQL snippets of code and results.

PostgreSQL Literate Programming with GNU Emacs
---
What is *literate programming*? [Literate Programming](https://en.wikipedia.org/wiki/Literate_programming){:target="_blank"} is a programming paradigm that makes you write a program in a more natural language, interleaving documentation and code together.
<br/>
GNU Emacs allows literate programming by means of *Org Mode* and its module *Org Babel*.
<br/>
I am already used to Org Mode, and I am already writing my own documentation, slides and papers with this great tool. But Org Babel can do much more for me: as you probably know I write several articles, papers, presentation for training events all related to PostgreSQL.
<br/>
The classical workflow is:
- write a slide or piece of document;
- execute an SQL statement (e.g. in a terminal);
- copy and paste the SQL statement into your slide or document;
- copy and paste the result into your slide or document.
<br/>
One huge problem about the above is that every time you change the initial statement, you have to repeat the process copy and pasting the results, and this can lead to errors, inconsistencies, and duty on yourself to keep the documentation up to date.
Moreover, imagine the output of a command changes from one version of PostgreSQL to another: you have to re-run every single command and repeat the copy and paste of the results.
<br/>
That's too much!
<bt/>
<br/>
Being BNU Emacs what it is, there's a much more smarter way to do it!

## Org Babel to the Rescue!
Org Babel is a module that allows Org Mode to execute a single snippet of code. The code is executed launching external processes, like interpreters (in the case of Perl, Python, etc.), shells or, in the case of our beloved database, `psql`.
<br/>
Let's see an example, imagine to write the documentation for a PostgreSQL transaction as follows:


<br/>
<br/>
```org
* An example of transaction

The following is a PostgreSQL explicit transaction:

#+begin_src sql :engine postgresql :dbhost miguel :dbuser luca :database emacsdb
BEGIN;

CREATE TABLE emacs( t text );

INSERT INTO emacs 
SELECT 'Foo' || v
FROM generate_series(1, 10);

COMMIT;
#+end_src

and when executed, the system replies with every command feedback:
```

<br/>
<br/>
For now, avoid the discussion about the connection parameters, that after all are quite easy to guess.
<br/>
If you place within the code block (i.e., in any poin from `#+begin_src` to `#+to_src`) and hit `C-c C-c`, Emacs will launch a `psql` connection to the database to execute the SQL set of statements. In other words, it will be like if you had manually typed the following on a command line:

<br/>
<br/>
```shell
echo 'BEGIN; CREATE TABLE emacs(t text); ...' |  psql -h miguel -U luca emacsdb 
```
<br/>
<br/>

The end result will be that your document automagically changes to:


```org
* An example of transaction

The following is a PostgreSQL explicit transaction:

#+begin_src sql :engine postgresql :dbhost miguel :dbuser luca :database emacsdb
BEGIN;

CREATE TABLE emacs( t text );

INSERT INTO emacs 
SELECT 'Foo' || v
FROM generate_series(1, 10);

COMMIT;
#+end_src

and when executed, the system replies with every command feedback:

#+RESULTS:
| BEGIN        |
|--------------|
| CREATE TABLE |
| INSERT 0 10  |
| COMMIT       |
```

<br/>
<br/>
that in turn, renders to something like the following

<br/>
<br/>
<center>
<img src="/images/posts/emacs/literate_postgresql_programming_1.png" />
</center>
<br/>
<br/>

Not bad, uh?

## Emacs and Org Babel Configuration

Emacs does not usually ship with Org Babel configured for SQL, so you have to place into your configuration file the following:

<br/>
<br/>
```lisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '((sql . t)))

(setq org-confirm-babel-evaluate nil)

```
<br/>
<br/>
The first three lines enables the SQL language, while the last one prevents Emacs to ask for confirmation before running every single snippet of code.


## Update the Results

In the case you change a snippet of code, you can simply re-issue `C-c C-c` to update consequently the results.

## Running All
Here it is the most fun part: imagine your documentation or slides include several snippets of code, and you want to update all the code results. Remember, you are in Emacs, and there must be a way to do it.
And in fact, you can run `C-c C-v b` to create and/or update all the result sections.
<br/>
This is particular handy for me when I want to update results based on a different version of PostgreSQL.


## Connection Parameters

As you have probably guessed, those parameters after the `sql` tag in the header of the code snippets tell Emacs how to reach the PostgreSQL server. In particular:
- `dbhost` is the remote hostname, with `localhost` for a local connection;
- `dbuser` is the database username
- `dbpasswd` is the user password, in clear text (!);
- `database` is the name of the database to which you need to connect to.


## Do not Repeat Yourself

You don't have to specify the connection properties on the header of every single piece of code: you can group properties in an Org Mode tree to handle all at once.
<br/>
Allow me to explain with an example document:

<br/>
<br/>
```org
* My experiments

#+begin_src sql :engine postgresql :dbhost miguel :dbuser luca :database emacsdb                                                     
BEGIN;                                                                                                                                  
CREATE TABLE emacs( pk serial, t text );                                                                                                
INSERT INTO emacs(t) SELECT 'Foo' || v                                                                                                  
FROM generate_series(1,10) v;                                                                                                           
COMMIT;                                                                                                                                 
#+end_src                                                                                                                                

#+begin_src sql :engine postgresql :dbhost miguel :dbuser luca :database emacsdb                                                     
SELECT * FROM emacs                                                                                                                     
LIMIT 2;                                                                                                                                
#+end_src
```

<br/>
<br/>

the above can be replaced with a more compact version like

<br/>
<br/>
```org
* My experiments
:PROPERTIES:
:header-args: sql :engine postgresql :dbhost localhost  :dbuser luca  :database emacsdb
:END:

#+begin_src sql 
BEGIN;                                                                                                                                  
CREATE TABLE emacs( pk serial, t text );                                                                                                
INSERT INTO emacs(t) SELECT 'Foo' || v                                                                                                  
FROM generate_series(1,10) v;                                                                                                           
COMMIT;                                                                                                                                 
#+end_src                                                                                                                                

#+begin_src sql 
SELECT * FROM emacs                                                                                                                     
LIMIT 2;                                                                                                                                
#+end_src
```
<br/>
<br/>

It is now possible to change in and manage the connection properties in a single place, so that if I, for example, need to change the hostname I can change on the `header-args` line and execute `C-c C-v b` to get all the require results.


## Give me the Shell, Quick!

Org Babel can, of course, execute and evaluate different snippets of code and languages. This allows you to insert into your own documentation not only SQL statements, but also maintaance commands to run thru the shell, like `service postgresql restart`.
And you can also execute directly `psql` as follows:

<br/>
<br/>
```org
#+begin_src shell                                                                                                                       
psql -h localhost -U luca -c 'SELECT t FROM emacs LIMIT 2' emacsdb                                                                      
#+end_src                                                                                                                               

#+RESULTS:
| t      |       |
| ------ |       |
| Foo1   |       |
| Foo2   |       |
| (2     | rows) |

```

<br/>
Please note that, since in Org Mode a `<TAB>` is used in conjunction with a table, the output is rendered as a two columns table even if you selected a single column.
<br/>
Remember that in order to allow Org Babel to evaluate the shell commands you need to enable the shell language in the Emacs configuration, therefore in your `.emacs` file you must now have something like:

<br/>
```lisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '(
   (shell . t)
   (sql . t)
   ) )

```
<br/>


# Conclusions

Emacs is a great tool! You can improve your PostgreSQL documentation by means of Org Mode and Org Babel.
<br/>
There is much more about the Org Babel, and this is just a quick introduction to let you taste the power of Emacs!
