---
title: "Dolt is git for databases"
tags: programming
date: 2021-03-23
---

I came across [Dolt](https://github.com/dolthub/dolt#dolt) which purports to be *"Git for Data"*:

> Dolt is a SQL database that you can fork, clone, branch, merge, push and pull just like a git repository. Connect to Dolt just like any MySQL database to run queries or update the data using SQL commands. Use the command line interface to import CSV files, commit your changes, push them to a remote, or merge your teammate's changes.

I haven't tried this, so I've no idea how well it delivers on the promise. However, this could be something of a game changer for [postgrest](https://postgrest.org/en/v7.0.0/). Postgrest is a webserver, that inspects your database and then exposes it as a Rest API:

> PostgREST is a standalone web server that turns your PostgreSQL database directly into a RESTful API. The structural constraints and permissions in the database determine the API endpoints and operations.

To begin with this sounds a bit nuts, you have to do your backend coding in SQL.  However, I've tried this, and I found the experience pretty nice. It mostly feels like coding in a functional language. It completely solves the [challenges of mapping object-relational mappings](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping#Challenges) otherwise known as the [Vietnam of Computer Science](https://blog.codinghorror.com/object-relational-mapping-is-the-vietnam-of-computer-science/). It does this because there is no mapping to even consider because you're already developing in SQL.

I found that using PostgREST meant that you were very aware of how your data was being stored, in contrast to using some ORM in which it's quite likely that you do not really know how your data is stored, and perhaps more seriously, you do not know how that data is marshalled from the database into your programming language's memory model. It's not that you cannot know, it's just that you have to dig to find that out, so day-to-day it tends to be ignored, until there is a problem.

So anyway, postgREST was pretty nice, with one fairly major caveat, it doesn't play very nicely with source code control. You can use both a database and, say git. But it's a bit painful, and it feels like you're fighting the system. What it means is you're writing scripts to migrate the database, rather than just writing code. So I found it pretty awkward.

Dolt, however, could change this and make the whole experience more pleasant. I stress though that I haven't tried Dolt at all, let alone in combination with postgREST. One little corollary if it did turn out to be a game-changer. The folks building postgREST, never knew that Dolt was going to come along, their philosophy was always challenging, but they simply trusted in their idea and built the part of the chain that they could. I think this is how innovation often happens. 
