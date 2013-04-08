---
author: admin
comments: true
date: 2008-03-28 13:46:34
layout: post
slug: updates-and-s
title: Updates and ::'s
wordpress_id: 34
categories:
- Updates
tags:
- Bugs
- PHP
---

In my previous article ([Microsoft Access and PHP](http://www.toosweettobesour.com/2008/03/24/microsoft-access-and-php/)), I discussed getting mdbtools setup and dynamically interacting with a instance of mdb-sql. Well it turns out somewhere in the pipeline (probably mdb-tools), we're either not getting an EOF or reading an EOF right and as such I can read lines from the program's STDOUT stream, but as soon as I hit the end of the output my script hangs indefinitely waiting for something and all my attempts at detecting an EOF failed. Sorry to disappoint.

On lighter news, my coworker got the greatest error ever:



> Parse error: syntax error, unexpected T_PAAMAYIM_NEKUDOTAYIM



Turns out according to [these guys](http://www.webmasterworld.com/forum88/5127.htm) it's Hebrew and refers to the double colon (::) operator.

Right...
