---

comments: true
date: 2008-02-02 01:47:41
layout: post
slug: plesk-open_basedir-workaround
title: Plesk open_basedir Workaround
wordpress_id: 20
categories:
- Software
tags:
- open_basedir
- Plesk
---

Unfortunately, Plesk does (and according to atomicturtle will continue to) hate on developers by enabling open_basedir by default and not give any way to change such behavior.

A work around is possible, overriding the setting in a vhost.conf file (httpd.include files are not so good because Plesk rewrites them every time a change is made to a domain's settings, or whenever Plesk feels like it). However, adding the line `php_admin_value open_basedir none` is completely ineffective (as I would assume the vhost.conf is being included before offending open_basedir changes are made?).

However I found that doing this (those familiar with Plesk's httpd.include will recognize this block):


```apache
<Directory /path/to/domain/httpdocs>
    <IfModule sapi_apache2.c>
        php_admin_value open_basedir none
    </IfModule>
    <IfModule mod_php5.c>
        php_admin_value open_basedir none
    </IfModule>
</Directory>
```


Guarantees that open_basedir will plague you no more.

Some may ask why I want open_basedir enabled, it's leaving me open to vulnerabilities! Some applications have a control panel on a separate sub domain and require the ability to write files to it's parent domain, something Plesk beats down with an ugly stick, as well as some applications requiring access to the system wide PEAR libraries, something open_basedir ironicaly beats down as well (despite the system wide PEAR libraries being included in the include_path directive).

Later I'll be digging through Plesk's ability to trigger commands on updates and domain creations to automate this vhost.conf setting (namely grabbing that path to the httpdocs folder that is so critical and time consuming).

**EDIT:** To apply the changes, follow markus' example:

> I just have my setting in the vhost.conf, and it works just fine. Just remember to update Plesks settings manually afterwards, with `/usr/local/psa/admin/sbin/websrvmng -u –vhost-name=example.com` (it doesn’t read directly from the vhost.conf).




