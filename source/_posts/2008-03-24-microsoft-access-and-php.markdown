---
author: admin
comments: true
date: 2008-03-24 23:32:11
layout: post
slug: microsoft-access-and-php
title: Microsoft Access and PHP
wordpress_id: 33
categories:
- Development
tags:
- Access
- PHP
---

Microsoft products can really be a big wrench in the finely oiled machine that is your development environment. While Microsoft does offer some excellent products and development tools, when you work from an open source context, these great tools become great hazards and headaches.

Case in point, access databases. Take as a recent example, a client that gets its data from a third party. That data comes in a Access (.mdb) format, probably convenient for the producing company as well as the assumption that chances are very likely that the person receiving the data will be able to read it (due to the widespread availability of Microsoft Office).

Importing said .mdb in PHP is not so nice or convenient, that is if you aren't running your server on Windows (which chances are you aren't). In Windows you merely need to either create an ADO object (or use [ADODB](http://adodb.sourceforge.net/)), or use COM to communicate with a local install of Microsoft Office to trigger an export. In Linux, neither of those choices are even available. However we thankfully have some good command line utilities, for example: [MDB Tools](http://mdbtools.sourceforge.net/). (Tip: Check out a copy from CVS if 0.5 is segfaulting on your database)

In addition to some unixODBC drivers, it provides some very nice command line utilities like mdb-tables (which prints the tables in a .mdb file), mdb-schema (which prints the schema for all or a specific table), mdb-export (which produces a CSV output of a specific table in the .mdb), and mdb-sql (which provides a subset of SQL interface to the .mdb file). All of these tools come together to form a very nice tool set to import a .mdb file.

Fun things however, as I was working on my import script (I was just using simple exec() commands and getting full output and parsing from there), I decided to play around with stream programming and provide an interface to mdb-sql using proc_open() to make the import less memory intensive. I ran into some snags using fgets() on the STDIN stream and when I solve those I'll post my code and a tutorial.
