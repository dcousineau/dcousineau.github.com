---

comments: true
date: 2008-07-28 09:52:18
layout: post
slug: are-we-still-arguing-the-merits-of-code-comments
title: Are we still arguing the merits of code comments?
wordpress_id: 75
categories:
- Opinions
tags:
- Comments
- Flame
---

So I see a link on reddit today to an article entitled [No, your code is not so great that it doesn’t need comments](http://www.reddit.com/comments/6tqre/no_your_code_is_not_so_great_that_it_doesnt_need/). The article site was down so I started reading the comments and was very disheartened to see an actual debate over whether pro developers put comments in or left them out.

For example, we have this from "ckwop":



> I find that the number of comments a developer puts in to their code is inversely proportional to the amount of experience they have.

I believe the reason for this is two fold.

Firstly, as you become more experienced you become fluent in reading code. You're not expending mental energy trying to grok the syntax and the flow of the program. It just comes naturally.

Secondly, as you become more experienced, you learn how to layout you code in a more readable way. You name classes and variables such that the code is, for the most part, self-documenting.



I don't quite know how to describe the absolute FAIL in this particular statement. While he does go on to say:



> The only comments left in programs written by experienced programmers are comments which document decisions. Why one approach was taken over another, for example.



It still doesn't redeem himself. Comments like his betray a staggering amount of arrogance that poisons the mind of developers and turn otherwise good programmers into spaghetti-spewing monsters that take on government jobs so it'll take longer for them to get fired.

This person (and sadly many others) assert that well written code is intuitive. These people are the same people that try and tell me Python is perfect for everything and I shouldn't work in any other language unless I'm working in the embedded space, in which case I should write myself a Python-To-Assembly script prefixed with the cute little "py".

Comments are meant to provide a quick overview of code flow and reasoning behind particular decisions. Comments should be clear and concise and allow a developer to spend 5 minutes figuring out where in the code base they need to be to fix the rounding issue rather than spending 40 minutes tracing program flow.

Comments are meant to help developers who either have limited experience in a language, who are new to a language, or who haven't used a particular language in quite some time. Your Haskell code may be clear and simple to you, but I can't read a single damned line of it: comment it!

Comments are meant for the developer who puts down a project to focus on something else for a few months only to pick the original code back up. The sign of a mature developer is one who accepts the fact that when he puts down code for a few days, he forgets the "why". No developer is immune to this, and anyone that tries to tell you they are different is not worth the time you wasted listening to them.

So, to "ckwop" and all the other developers out there that think code should speak for itself I have a few things to say:


* If code could speak for itself, it wouldn't be called code
* You will not remember your design decisions, and in some cases even code flow, after a month
* Comments make it much easier for me to debug when your company hires me to fix your failures both as a programmer and as a human being.

In closing: The debate has long been over, comment your fucking code.
