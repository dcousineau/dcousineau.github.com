---

comments: true
date: 2009-02-13 01:58:30
layout: post
slug: php-mumbles-growl-and-dbus-sweeet
title: 'PHP, Mumbles (Growl), and DBus: Sweeet'
wordpress_id: 372
categories:
- Development
tags:
- DBus
- Growl
- Linux
- Mumbles
- PHP
- Ubuntu
---

So, after reading [Mark Shuttlework's blog on ideas for notifications in Ubuntu](http://www.markshuttleworth.com/archives/253) (basically mimicking [Growl](http://growl.info/) notifications for the Mac), I decided I wanted that kind of functionality, but... you know, NOW!

There are a multitude of options available but currently I'm liking [Mumbles](http://www.mumbles-project.org/). <del>Unlike, say, [Specto](http://specto.sourceforge.net/), which does the monitoring itself, Mumbles provides a DBus interface, a command-line app named `mumbles-send`, and (I'm not sure if it's implemented in the current stable download) libnotify support.</del> Woutc, from the Specto project, commented below explaining that Specto is not intended to be a Mumbles competitor, but a package to easily monitor system internals (I quote _"...the purposes from mumbles and specto are different…specto is monitoring what happens outside your desktop, mumbles monitors what happens on your desktop (or in your network)..."_) Apparently he has plans to build a Mumbles plugin so that one can optionally have Specto send its messages to Mumbles for display.

I decided the <del>best</del> easiest route is to access the internal DBus API, however the forums and other resources on the Mumbles site... well... just plain suck. And by suck I mean tell you that something exists and... thaaats about it.

{% img /images/posts/2009/02/d-feet.png %}

Well in my Google quest I discovered the existence of [D-Feet](https://fedorahosted.org/d-feet/), a DBus debugging tool (on [Ars](http://arstechnica.com/open-source/news/2008/01/learning-from-d-feet-a-quick-look-at-a-new-d-bus-debugger.ars) of all places). Thanks to a quick `sudo apt-get install d-feet` I found the existence of a public interface in `info.growl.Growl` that allows for a `Notify(title, message)` signal to be passed.

Well, with a place to access all I needed was DBus integration with PHP (because I want to start sending debug notification via Growl/Mumbles like [this idea](http://framework.zend.com/wiki/pages/viewpage.action?pageId=8454257) on the Zend Framework incubator).

Luckily, GREE Labs provides a [DBus C extension for PHP](http://labs.gree.jp/Top/OpenSource/DBus-en.html) that was easily downloaded and installed. Once installed I read [the documentation](http://labs.gree.jp/Top/OpenSource/DBus/Document-en.html) to get an idea on how to use the API (which is a 1:1 mapping to the [DBus API](http://dbus.freedesktop.org/doc/api/html/)).

After a bit of hacking I finally came up with an alpha product:


```php
<?php
function null_callback()
{
        var_dump(func_get_args());
}

$dbus = dbus_bus_get(DBUS_BUS_SESSION);

$m = new DBusMessage(DBUS_MESSAGE_TYPE_SIGNAL);

$m->setPath('/info/growl/Growl');
$m->setInterface('info.growl.Growl');
$m->setMember('Notify');
$m->appendArgs('Hello World!');
$m->appendArgs('Lorem Ipsum Dolor Sit Amet. (' . time() . ')');

$r= $dbus->send($m);
```


After running the script in the console, did I see success?

{% img /images/posts/2009/02/mumbles-php.png %}

Damn right I did!

<del>
Though with a caveat:
</del>

<del>
```text
Warning: dbusconnection::sendwithreplyandblock(): dbus_connection_send_with_reply_and_block() failed (Traceback (most recent call last):
  File "/var/lib/python-support/python2.5/dbus/service.py", line 643, in _message_cb
    (candidate_method, parent_method) = _method_lookup(self, method_name, interface_name)
  File "/var/lib/python-support/python2.5/dbus/service.py", line 244, in _method_lookup
    raise UnknownMethodException('%s is not a valid method of interface %s' % (method_name, dbus_interface))
UnknownMethodException: org.freedesktop.DBus.Error.UnknownMethod: Unknown method: Notify is not a valid method of interface info.growl.Growl
) in /home/daniel/test.php on line 21
```

```text
Call Stack:
    0.0002      64496   1. {main}() /home/daniel/test.php:0
    0.0024      66372   2. dbusconnection->sendwithreplyandblock() /home/daniel/test.php:21
```
</del>

<del>
Apparently Mumbles seems to be throwing an Exception that it's failing to to find the Notify signal (though everything works correctly). I guess I could ignore this, use the error suppressor (`@`), or even yell at GREE Labs to have the objects throw exceptions so I can catch them... I guess I'll hack on it some more and report back. I should probably spend more time learning the DBus specs since this is my first project playing with DBus...
</del>

**Update:** Though it's not listed on the API page for PHP DBus's API, there is a method `send()` that takes only a single argument (the message object).

**Edit:**

I saw a question on Reddit asking what is the use of this technique if it's limited to the desktop it was called on and PHP is primarily a server side language. Why not do this in Python or Perl?

Well, Mumbles was written in Python so there's no point in me doing this in Python: it's already been done.

However, the primary use case of a technique like this is having a web app post notifications and errors for the developer. When I work on a site I have a local copy running on my desktop and/or laptop, so when it posts Growl/Mumbles notifications, I get them on my desktop. It's great for situations where, say, I have a page in my PHP app that never displays its contents because it processes data then redirects to another page. If a warning or other non-fatal error that I should really fix occurs, then I would normally have to dig through the system wide PHP error log (if you even have error logging enabled). However, if I wrote a custom error handler that posts errors to Growl/Mumbles as they happen, when I visit a transitory page like I described I get little Growl/Mumbles notifications showing that I screwed up!
