---
author: admin
comments: true
date: 2008-09-12 10:16:31
layout: post
slug: weird-errors-and-xdebug-to-save-the-day
title: Weird errors and XDebug to save the day!
wordpress_id: 148
categories:
- Development
tags:
- Bugs
- PHP
- XDebug
---

I had an interesting (and by interesting I mean completely mindbogglingly frustrating) error that I cannot summarize.

To start things off I notice to my dismay visiting a specific page on my website would result in a white screen (as would be expected for a fatal PHP error with error_reporting turned off). Problem is I have the Zend Framework's ErrorHandler plugin setup to load up a view that just var_dumps the error(s) for the development process and obviously I'm not getting this.

Checking the error logs I see... an out of memory issue. Funny thing was no matter how much I upped the limit, the script would always exceed my limit.

So I disabled the ErrorHandler plugin, re triggered the error and checked the logs. Now nothing is showing up in the logs but the script is still white screening.

Using my trusty `var_dump` I started trudging my way through the code looking for the culprit, I first tried just var_dumping and print_ring the ErrorHandler's errors object and even the ErrorHandler instance itself, but any attempts resulted in my out of memory problem. There really wasn't a specific break point in the code as the code finished execution normally (just with an error, so not really normally eh?).

Eventually I finally just got down on my knees and var_dumped the response object from the Front Controller and the resulting output actually crashed Firefox! One of my initial suspicions, recursion problems, was correct but not in the way you'd think.

It turns out the exception object that was being thrown was either self referencing or something silly because any attempts to print the object resulted in an endless loop. You see, PHP's default var_dump and print_r commands recursively display objects and associative arrays without any depth limits. PHP doesn't even throw a recursion depth limit!

So I installed Xdebug to my live server ([an ordeal in and of itself](http://charlmatthee.blogspot.com/2008/06/installing-peclpear-php-modules-on-rhel.html)) I was finally able to view the contents of the output (thanks to Xdebug's depth limits) and lo and behold my error message! 



> SQLSTATE[42S22]: Column not found: 1054 Unknown column 'j.created_at' in 'field list'



Turns out the issue was with an exception that Doctrine threw.

Such a freaking let down.
