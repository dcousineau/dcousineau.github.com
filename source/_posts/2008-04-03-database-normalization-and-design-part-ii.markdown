---

comments: true
date: 2008-04-03 14:07:56
layout: post
slug: database-normalization-and-design-part-ii
title: 'Database Normalization and Design: Part II'
wordpress_id: 38
categories:
- Development
tags:
- Database Design
- MySQL
- Normalization
---

(This is the sequal to my previous post, [Database Normalization And Design: A Primer](/blog/2008/04/01/database-normalization-and-design-a-primer/))

<!-- more -->

Last time we covered creating the database for keeping track of our unique books. Now we continue on to the rest of the project requirements. If you remember:



> 
The application shall keep inventory on a library of technical reference **books**. **Users** shall be able to log in and request books for checkout. _Administrators_ shall be able to update the inventory and _approve_ checkouts as well as confirm returned books.




Users is fairly easy to implement. We have an autonumber primary key, ID, that we use to identify users. We need columns for a user's handle, hash. To identify administrative permissions we could actually implement a database store for fine grained ACL permissions, however since we're being fairly simple here we'll just want an unsigned integer to indicate our administration level. We'll default to 0 meaning the base permissions (just viewing and requesting check out), and any step up provides access to more and more permissions (in our case: 1 allows the management of the library and 2 symbolizes root access).

Where we really start to have questions is how do we want to handle checkouts and inventories? There is actually not a singular correct answer in this aspect. For example, someone might use an integer field in the Books table to keep track of the total number of those books. I however prefer to use a separate table to keep track of inventories. In this table, BooksInventory, we have a unique entry linking to a specific book for every copy of that book we have. So if we have 10 books, we have 10 rows in the BooksInventorytable each linking to the same book. This allows the Books table to be easily exportable (no application specific data is stored within), allows us to track data on individual copies of the book (e.g. numbered copies, condition on said copies, etc.), and allows us to work with inventories without having to worry about table level or row level locks on the Books table. Also, remember the concept that a single row in the Books table represents a singular book. The count of the copies of that book does not fit well in that model.

To handle checkout requests (requests must be approved before they are checked out) we add 2 fields to the BooksInventory table: UsersID (representing the ID of the user requesting checkout) and CheckedOut (a bit/boolean field representing approval of checkout or not). When we are selecting data, we can effectively consider all entries in BooksInventory that do not have CheckedOut set to true as in stock. We can also pull up a list of books pending approval of checkout by selecting all entries in BooksInventory where UsersID is not null, but CheckedOut is false.

When designing a system, however, one must keep in mind as many potential scenarios to work out bugs before they rear their ugly heads. An easy way is to think in terms of "instantaneous scale." What I mean by this is when designing a system, think to yourself "How would my system react if 2+ users performed X action [at the exact same time]." In terms of the BooksInventory table, this question becomes "What happens if 2+ users request to check out the last copy of a book?" Obviously, the last user to perform the checkout request will have their UsersID set, but this presents our bugs and problems. Usually, checkout requests are first come first served, so an easy remedy would be only allow checkouts on copies that do not have a UsersID set, however what if an administrator denys User A's request, but had they been able to see, would approve User B's request?

A solution could be another table called BooksRequests that contain UsersID's and request time. As a request is approved, the UsersID data is moved to the BooksInventory table to indicate checked out, which means our CheckedOut boolean is now not needed and can be removed.

Like I mentioned in the last post, examining a database design visually really helps cement concepts and ideas:

{% img /images/posts/2008/04/db_diagram_final.png 'Final Diagram' %}

Hopefully those that don't have a great grip on designing a database from scratch will have a good example to work from.
