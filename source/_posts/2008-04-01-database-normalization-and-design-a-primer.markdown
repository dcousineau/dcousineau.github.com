---

comments: true
date: 2008-04-01 13:34:36
layout: post
slug: database-normalization-and-design-a-primer
title: 'Database Normalization And Design: A Primer'
wordpress_id: 36
categories:
- Development
tags:
- Database Design
- MySQL
- Normalization
---

I am by no means a database expert, such titles are best left to people like [Peter Zaitsev and Vadim Tkachenko](http://www.mysqlperformanceblog.com/), however one of my co workers has been asking me for some help on how I come about my database designs, particularly issues concerning [normal form](http://en.wikipedia.org/wiki/Database_normalization).

Whlie I could spend several posts going over the intricacies of 1st, 2nd, 3rd, and 4th normal form as well as "The key, the whole key, and nothing but the key so help me Codd" Boyce-Codd normal form, however reality dictates that time spent strictly to these academic levels is either time wasted (projects with due dates unfortunately cannot spend forever on the database design) or pointless as being reasonably intelligent will bring you very close, if not within 3rd normal form.

If you're really interested, Wikipedia provides a great resource to get started learning the more academic aspects.

I'm going to assume that you have done a bit of database work and at least understand the concept of a [join](http://en.wikipedia.org/wiki/Join_(sql)).

The one of the goals of database normalization is to reduce the repetition of information and thereby reduce the complexity of an update of said information. If the same string is in the database N times, we should abstract it so that if we need to update said string due to a typo or anything else, we need to only change but one location.

<!-- more -->

Since I like to ground myself in reality and practicality, lets work with some real world examples. Lets assume our requirements or mission statement is this:



> 
The application shall keep inventory on a library of technical reference books. Users shall be able to log in and request books for checkout. Administrators shall be able to update the inventory and approve checkouts as well as confirm returned books.




Fairly simple, almost e-commerce, but more related to a simple project I have to do for class (oh how convenient). Before we continue, we need to examine these project requirements for the information we'll need for the database:



> 
The application shall keep inventory on a library of technical reference **books**. **Users** shall be able to log in and request books for checkout. _Administrators_ shall be able to update the inventory and _approve_ checkouts as well as confirm returned books.




From the outset we can identify 2 major data concepts, Books and Users, as well as mark states (in _italics_) for future reference.

Now, further contact with the client would be required to specify the exact information required for all of our tables, however for the sake of this post lets just go with the bare minimum (design gets weird when you start debating on how to store, say, ISBN numbers which have 2 major and a few minor formats, one 10 character and another 13). We all know that books have ISBN numbers, Titles, Authors, and Publishers, all of which are information we should definitely keep on hand. One not well versed in relational design would throw all of these attributes into their own columns, however we come across some issues.

One of the first steps is identifying your primary key of the database. For the uninformed, a primary key is a column (or columns) that uniquely identifies a single role. Meaning, if I have a value for the primary key, it will return 1 and only 1 row each and every single time. There can be NO ambiguity (e.g. a primary key of (1) returning 2 rows, how do we know what row we really wanted?). Upon first glance the obvious choice would be to set our ISBN number as our primary key as we all know that ISBN numbers are distributed to books on a 1:1 basis. However there's a potential pitfall. If we've joined all of our tables on the ISBN, what happens when you need to alter a book's ISBN (possibly due to a typo discovered late)? In this, and most other cases, our best bet is to use a field unrelated to the ISBN, such as an auto-numbered INT field.

Lets assume we have 12 books on programming languages, all written by 2 different authors (some books have one other, some the other, and some have both), but all published by a single publisher. If we were to look at a database dump there would be a large amount of repeated data.

Since each book can only have 1 publisher, we can move any information pertaining to the publisher (name, address), into it's own table and link the two via a unique auto-numbered INT field. Now books will contain a field called publishers_id that will contain the ID number of the publisher who published the book.

Since each book can have multiple authors, we cannot just use the 1:1 relationship that the "Books Have Publishers" relationship above is. We need an intermediary (n:m) table, call it say "BooksHaveAuthors". The primary key for this table is not a single column however, its two columns: a column linking to a book and a column linking to an author. Now when we perform our queries in our applications, we can pull all the authors of a book via a query selecting all rows where books_id is X, or we can select all the books an author has via the opposite.

Now, if you're smart you are using a visual designer such as [MySQL Workbench](http://dev.mysql.com/workbench/), you'll have a visual such as this (this example is using MSSQL's database diagramming utility):

{% img /images/posts/2008/04/books_dia.png 'Books Diagram' %}

Make a lot of sense when you look at it visually?

([Part II](/blog/2008/04/03/database-normalization-and-design-part-ii/))
