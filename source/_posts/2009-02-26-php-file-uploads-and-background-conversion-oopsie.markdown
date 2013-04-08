---
author: admin
comments: true
date: 2009-02-26 15:53:19
layout: post
slug: php-file-uploads-and-background-conversion-oopsie
title: 'PHP File Uploads And Background Conversion: Oopsie'
wordpress_id: 393
categories:
- Development
tags:
- File Upload
- PHP
---

So I spent an inordinate amount of time tracking down a bug recently for a former employer in a system that accepts media file uploads and converts them on the fly.

The system spawns off a small script that manages mplayer/ffmpeg/lame into the background and holds on to the PID to track the conversion process. HOWEVER, there was a problem in the system where large files were being ignored and no error messages were popping up.

What ended up happening was I forgot to take into account PHP's file upload behavior in the conversion process.

When PHP accepts a file upload it creates a temporary file (usually in `/tmp/`). The caveat is PHP frees (read: deletes) the temporary uploaded file at the end of script execution.

So, while the converters could convert a small file in the same encoding as the desired output, a large file (say, a 10mb MPEG) that needed to be converted to a different format (say, a FLV), the converter would begin working in the background but when the PHP script finished executing, PHP would delete the file right out from under the converter's feet.

So, a tip to anyone doing anything similar, go ahead and at least rename/move the file (I created a unique MD5 hash from the current time plus a few other things and use that as the new file name) and pass that to any background processing functions so PHP won't delete the file at the end of processing.

It will save a WHOLE lot of headaches for you in the future.

Also, if anyone is interested, I will probably write up a tutorial on how to write a PHP script that will convert your MP3's in the background.
