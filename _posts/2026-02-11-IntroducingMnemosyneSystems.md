---
layout: post
title:  "Introducing mnemosyne systems"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A new entity in the PostgreSQL landscape.

# Introducing mnemosyne systems

The last week a new entity appeared in the PostgreSQL landscape: **[mnemosyne systems](https://www.mnemosyne-systems.ai/){:target="_blank"} **, the last creation of [Jesper Pedersen](https://github.com/jesperpedersen){:target="_blank"}.

The initial page says it all:

<br/>
<br/>
```
mnemosyne systems create intelligent solutions for PostgreSQL that allows you to achieve your disaster recovery (DR) and high-availability (HA) goals.
```
<br/>
<br/>


What are those *intelligent solutions* supposed to be?
<br/>
If you take a look at [the products overview](https://www.mnemosyne-systems.ai/products/overview/){:target="_blank"} you will see that the foundation is based on **pgagroal** (a fast and reliable connection pooler), **pgmoneta** (a backup/restore solution) and **pgexporter** (a Prometheus metrics handler).
All together, these solutions, give you the capability to provide High Availability (HA) and Disaster Recovery (DR) to your PostgreSQL clusters. The keypoint is to achieve, as much as possible, a whole platform where human intervention can be narrowed and *Recovery Time Objective (RTO)* and *Recovery Point Objective (RPO)* are as lower as possible, keeping in mind that having both set to zero could be impossible, especially in large scale scenarios (e.g., petabyte databases).


All the foundation, i.e., the above mentioned products, are all   **[Open Source Software](https://www.mnemosyne-systems.ai/opensource/why/){:target="_blank"}**, a development approach that Mnemosyne Systems encourage from the very beginning. Quite frankly, I do believe that Open Source is the only good way of developing software.


The [team](https://www.mnemosyne-systems.ai/about/team/){:target="_blank"}, including yours truly, is made by very smart people and is going to grow in the near future. Being a member of this team, I can truly say I learnt a lot from other members and I really believe this has been so far an exciting and very useful experience.

It is worth having a look at  the *vision* of this new entity, which is quite well explained by its founder Jesper in [this page](https://www.mnemosyne-systems.ai/about/vision/){:target="_blank"}: `to create a complete solution [...] nd work with the Open Source community [...]`.

Last but not least, in case you are thinking about, let's learn who [Mnemosyne was](https://en.wikipedia.org/wiki/Mnemosyne){:target="_blank"}.
