---
author: admin
comments: true
date: 2009-01-16 14:47:13
layout: post
slug: oss-bugs-are-features
title: OSS Bugs Are Features
wordpress_id: 259
categories:
- Opinions
tags:
- Bugs
- Community
- OSS
---

I've been thinking a lot lately about a presentation given by [whurley](http://whurley.com/) at a [refreshBCS](http://refreshbcs.org/) meeting a long time ago. He gave his presentation on Open Source Software, only SLIGHTLY an era of his expertise (just in case you can't tell, that was sarcasm) and raise an interesting point.

He talked about how when [BMC Software](http://www.bmc.com/BMC/Common/CDA/hou_Page_Detail/0,3464,9926222_10636326_76443375,00.html) picked him up to head up their open source efforts, he battled a little bit the developers. They wanted to make sure the code was perfected, polished, and at its best whenever they started dumping it on the public. Of course they'd want to do this, releasing code as OSS is almost equivalent to deciding to present yourself to the work naked and without makeup. They wanted their friends who could see the code to be in awe, and their code base to be a shining example of a project done right.

Whurley wanted them just to release it. Not because he wanted it now, not because of impatience or deadlines or anything else. He wanted them to just release it because, and I'm paraphrasing him, releasing open source thats a little buggy will actually help grow and foster the project. Granted, you do not introduce bugs, you just open and display the code and start making releases even though there are still tickets in your bug tracking system.

He gave the hypothetical example of the stereotypical arrogant OSS developer finding a bug in BMC's FooBar software, calling the BMC devs dumbasses, fixing it (because it was a small and easy bug), and now being invested in the software. You don't really lose much reputation, but you gain an invested and loyal following.

While this sounds good and true, and it makes a lot of sense (if there's nothing to do, OSS developers will never get involved), it never really clicked for me until I associated another story with whurley's principle.

I meet Elizabeth Marie Smith (aka auroraeosrose) at ZendCon08 and had good times talking with her and many of the other #phpc members. At one point in the conference, at an opening for one of her talks (a brief overview of PECL extensions), she told the audience how she got into programming PHP and PHP C extensions. She wasn't a computer science major, she only started in the early 2000's (prior to that not much computer science experience).

What had happened was, to the best of my recollection and [her seven things post](http://elizabethmariesmith.com/2009/01/seven-weird-things/), in the early days of 2001-ish she was really big into Anime and wanted to start a fan site. Her dad helped her install a forum (written in PHP) on a server he was running out of his house on a cable connection.

Well, as with many of the forum projects, there were bugs to be had in the software. Elizabeth was dedicated to the idea of her site and decided to break open the forum and dig around to fix the bugs. Fix them she did but everything kinda snowballed from there. In fact it was bugs in the PHP-GTK extension that caused her to learn C (not an easy feat) and become the maintainer of PHP-GTK as well as the Windows port of PHP maintainer.

It was bugs and problems in the software that caused her to be where she is now, it was bugs in the OSS world that caused her to latch on to PHP-GTK and the Windows PHP port and get them to where they are today. Though she told this story a long time ago, and whurley's presentation was even further back yet, it only just now clicked that I had seen a real life example of whurley's principle of "just publish it, flaws and all."

The lesson from all this can even be applied outside of OSS development. It's a lesson on perfectionism and can be applied towards my personal development practices. I know that agile teaches to publish early and publish often, fixing bugs and adding features on the way, but perfectionism still plagues me. I have maybe 10 drafts on this blog on small little PHP and Python topics I could publish that I never do because I feel it's not good enough, or too simple. I have had code at work that I could have released a week ago but instead I obsess over making sure its "perfect" when a simple staging deployment will keep everyone happy, keep my moral up, and show me where the REAL bugs are.
