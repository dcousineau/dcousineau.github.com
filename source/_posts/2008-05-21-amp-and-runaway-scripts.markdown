---

comments: true
date: 2008-05-21 00:56:07
layout: post
slug: amp-and-runaway-scripts
title: '*AMP and Runaway Scripts'
wordpress_id: 53
categories:
- Development
tags:
- Linux
- PHP
- Processes
---

Peter Zaitsev [posted](http://www.mysqlperformanceblog.com/2008/05/20/apache-php-mysql-and-runaway-scripts/) a very interesting test on how PHP and Apache handle runaway PHP scripts.

I'm sure all of us have had a long executing SQL script or at least a runaway script, and he points out even with [`ignore_user_abort`](http://www.php.net/manual/en/features.connection-handling.php) set to FALSE and max execution times set both in PHP and Apache, a runaway SQL query can execute beyond the timeouts, even when aborted by the user (stop button on the browser).

Even using the function [`connection_aborted()`](http://www.php.net/connection_aborted) to check for a user abort fails.

However, the script will cease on a user abort if it performs regular `ob_flush(); flush();`  commands. For example:


```php
<?php
echo("Hello");
for($i=0;$i<10000;$i++)
{
	sleep(1);
	echo('.');
	ob_flush();
	flush();
}
```


Just one more reason why I love the [MySql Performance Blog](http://www.mysqlperformanceblog.com/).
