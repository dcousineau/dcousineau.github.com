---
author: admin
comments: true
date: 2011-01-29 13:58:56
layout: post
slug: a-lesson-in-good-architecture
title: A Lesson In Good Architecture
wordpress_id: 487
categories:
- Software
---

We recently began working on our biggest event for our biggest client. In the process of getting everything up and running before the event started, we noticed a show-stopper bug appear out of nowhere.

Our media processor when finished converting videos and creating thumbnails of images publishes said files to our CDN. Only a few hours before our event and our CDN was failing probably 80% of the SSL handshakes for our requests to publish files. There is no need to name names as it was a difficult and obscure bug to diagnose in an extremely short amount of time. We needed to get it working, our CDN provider wasn't going to be able to resolve the problem in the extremely short timeframe we required, so the decision was clear: we needed to switch providers for the new content.

Normally this is a 2 week job, switching API calls, testing, finding all the occurrences (we process both videos and photos in different styles of batches), etc. We were able to complete it and have it live in under 1.5 hours. Why? We (and really I mean my coworker as this decision was made before I came on) made som really damned good architecture decisions. Primarly:




	
  1. We used OSS, primarily [Zend Framework]() which had well written API libraries for our new CDN

	
  2. All file system operations were abstracted to adapters that all inherited a "FileStore" base class



So, create a file store for our new CDN using the well written, off the shelf API. Change 4 lines of XML configuration, deploy, and we were in business.

Had my coworker succumbed to laziness or listen to some people who claim "oh you're just making it too complicated" or "just get it up quickly, damn the design decisions" we would have been sunk. Instead he took the time to utilize the adapter pattern (even though it probably added an extra 2 days to his total coding time) despite all signs pointing to us never needing to switch CDN's (and certainly not within a 2 hour timeframe).

So let this be a lesson to us all. Abstraction and design patterns sometimes feel like an "enterprisey overcomplication" but they aren't there for everyday needs. They exist for those days when a service provider gets hit by an obscure bug and you have to swap out a component under extreme time schedules. So suck it up, use them, and thank yourself when you find yourself in such a situation.
