---
author: admin
comments: true
date: 2009-03-10 08:30:23
layout: post
slug: calculating-daylight-savings-time-boundary-in-php
title: Calculating Daylight Savings Time Boundary In PHP
wordpress_id: 397
categories:
- Development
tags:
- Daylight Savings Time
- PHP
- strtotime
---

I had an issue recently where I needed to calculate the Unix timestamp for the daylight savings time boundaries. According to the [United States Naval Observatory](http://aa.usno.navy.mil/faq/docs/daylight_time.php), daylight savings time begins the **Second Sunday of March** and ends on the **First Sunday of November**.

Awkward date calculations if you don't have the magical [`strtotime()`](http://php.net/strtotime) function in PHP. `strtotime()` is able to do relativistic time conversions from the common "+1 hours" to the more complex (and more relevant) "Second Sunday March 0".

Why do we do the 'March 0' in the above string to calculate the start boundary? Doing some tests with `strtotime()` reveals some behavior you should be aware of.

`strtotime("March");` doesn't actually give you the first of March, it gives you the current day in the month specified (`date('D, F j, Y, g:i a', strtotime("March"));` returns `Tue, March 10, 2009, 12:00 am` at the time of this post). Doing a similar test but substituting April for March results in the 10th of April (at 12 am).

Now in most cases doing a `strtotime("March 1")` will suffice (`date('D, F j, Y, g:i a', strtotime("March 1"));` results in `Sun, March 1, 2009, 12:00 am`). However, as you can tell, this month is going to be awkward because the first day of the month is a Sunday. `strtotime()`, when calculating the fuzzy "Second Sunday," doesn't include the current day. So calculating `date('D, F j, Y, g:i a', strtotime("Second Sunday March 1"));` will actually return the 3rd Sunday (`Sun, March 15, 2009, 12:00 am`).

So our solution is actually to take a step back and return the **PREVIOUS** day to the first day of March and then calculate the Second Sunday from there (since we know that starting from the next day will account for the first). Normally, dealing with the last day in February is a headache, BUT again thanks to PHP magic we don't have to figure out if it's February 28th or 29th, we merely do a "March 0" which steps us back a month (`date('D, F j, Y, g:i a', strtotime("March 0"));` returns `Sat, February 28, 2009, 12:00 am`). From there we can calculate the Second Sunday with ease and eventually determine that the second sunday is March 8th, which is confirmed by the above USNO website.

Applying these lessons to calculating the first Sunday in November we come up with the following snippit:


```php
<?php
$remove_hour = strtotime("Second Sunday March 0");
$add_hour = strtotime("First Sunday November 0");

$time  = time();

if( $time >= $remove_hour && $time < $add_hour )
{
    var_dump("Lost an hour");
}
else
{
    var_dump("Gained an hour");
}
```


Cheers!
