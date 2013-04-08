---

comments: true
date: 2008-07-23 10:59:36
layout: post
slug: net-perspective-cross-post-time-is-money-save-time-with-automation-and-phing
title: '[Net Perspective Cross Post] Time is Money: Save Time With Automation And
  Phing'
wordpress_id: 69
categories:
- Development
tags:
- DBDeploy
- Deployment
- Phing
- Project Management
- rsync
- Xinc
---

**(Originally posted on [my company's blog](http://blog.net-perspective.com/2008/07/23/time-is-money-save-time-with-automation/))**



I've been talking a lot about automation as of late which led to my [posting](http://www.toosweettobesour.com/2008/05/15/automating-the-development-workflow/) a generic, overload broad view of the tools and utilities I use here at Net Perspective. However, I wanted to go into detail about my real world uses of a particular tool, [Phing](http://phing.info/).



<!-- more -->

In my time with [Net Perspective](http://www.net-perspective.com/) (and our previous existence), I've completely dreaded live deployment days because yes, they were full days. An entire day dedicated to waiting for FTP uploades to finish, chmod'ing directories (because Plesk hates me), setting up databases, and testing. All of this despite the fact that we do periodic staging deployments on the same server!

I guess a lot of thanks goes out to [Pro PHP: Patterns, Frameworks, Testing, and More](http://www.amazon.com/Pro-PHP-Patterns-Frameworks-Testing/dp/1590598199) by Kevin McArthur. While this book felt more like a Zend Framework tutorial, it did provide a brief introduction to two utilities I use here at work. One of which we care is our focus for the moment: [Phing](http://phing.info/).

Phing is a build environment based on Ant which should be familiar to those in Java world. For those not familiar, Phing uses XML based build files with tags representing tasks and task groupings. At Net Perspective, I use Phing build scripts for everything from packaging up tarballs of utilities we've published (simple jQuery plug-ins, etc.) to _automating the live deployment of client sites_. I'm particularly happy with the latter as my deployment time has been reduced from a few hours to between 10 minutes max (for a brand new, fresh upload) and about 30 seconds (for an update).

Basic Phing usage is pretty self explanatory, however you may wish to search Google for Phing tutorials or read my post linked above.

Furthermore, my build scripts make use of custom command executions requiring both the execution environment to be *NIX based and for you to read [this article](http://troy.jdmz.net/rsync/index.html) on using rsync over ssh without a password (using pre-shared keys).



### Setup



I start out with 2 files: **build.properties** and **build-live.xml**. I separate live deployment from the default build.xml Phing uses as, even though I harp on automation, live deployments should be manually triggered.

The file build.properties will be our configuration. If you set it up right, you can use the same script for all your projects as long as you alter the build.properties file accordingly. In it I create a few keys that will be necessary.


```ini
copy.live.dir=/path/to/site/on/remote/machine

copy.live.chmod="${copy.live.dir}/any/directory"

copy.live.chown=user
copy.live.chgrp=group

db.live.host=localhost
db.live.user=localuser
db.live.pass=localpass
db.live.name=localdb

progs.mysql=/usr/bin/mysql
```


You'll notice the properties copy.live.chown and copy.live.chgrp. A problem I encountered is Apache tends to get finicky about web file ownership, so I set this property to whatever user and group would be set after using FTP to upload. This way the final results of the build script will look as if you used FTP (but you didn't, because you're smarter than that).

With my build.properties set, I'll be begin with my build-live.xml. I start out with the basics:


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="Local Project" default="dist" basedir=".">
       <tstamp />
<property file="./build.properties" />
</project>
```


The **<tstamp />** initializes the ${DSTAMP} and ${TSTAMP} properties and the **<property file="./build.properties" />** obviously includes our build.properties file.

Continuing along I create my default build target **dist** to just reference a few external targets:


```xml
<target name="dist" depends="copy,clean,chmod">
</target>
```




### Upload



Now we begin the fun part, assuming you've already followed the link above about setting up a pre-shared ssh key, we can create our copy target which will be responsible for uploading new/changed files to the server:


```xml
<target name="copy">
       <exec command="rsync -aCz -e 'ssh -i /path/to/ssh/preshared/key' ${project.basedir}/path/to/local/files remoteuser@remoteserver:${copy.live.dir}/httpdocs/" escape="false" />
</target>
```


The key awesomeness is obviously our rsync command. Our particular variant contains the -C flag which ignores CVS/SVN type files (in particular, .svn folders) and omits the -v verbosity flag.



### Cleanup



Next we need to build a clean target that will clear any compiled/cache directories, set ownerships, and generally clean house:


```xml
<target name="clean">
       <echo msg="Clearing Compiled..." />
       <exec command="ssh -i /path/to/ssh/preshared/key remoteuser@remotehost rm -rf ${copy.live.dir}/httpdocs/delete/contents/*" escape="false" />
       <echo msg="Changing PROJECT_STATUS to live..." />
       <exec command="ssh -i /path/to/ssh/preshared/key remoteuser@remotehost &quot;sed -r -i 's/define\(\W*''PROJECT_STATUS''\W*,\W*''(.+?)''\W*\);/define\( '\''PROJECT_STATUS'\'', '\''live'\'' \);/g' ${copy.proof.admin.dir}/path/to/config.file.php&quot;" escape="false" />
</target>
```


Here in our first exec task, rather than running rsync, we use ssh and our pre-shared key to run a command on our remote machine. In this case an rm -rf to clear our a particular directory (could be a /cache, /compiled, depends on your setup). In our second exec task you'll notice some uglies involving parenthesis escaping, however what we are essentially doing is running _sed_ on the remote machine to flip a configuration value. What we do is maintain the configuration for all stages of deployment so that to change between the 2 or 3 stages it is just a single constant being changed.



### Ownership



Finally we need to set ownerships of all the files so that Apache will play nice:


```xml
<target name="chmod">
       <echo msg="Setting Ownership..." />
       <exec command="ssh -i /path/to/ssh/preshared/key remoteuser@remotehost chown -R ${copy.live.chown}:${copy.live.chgrp} ${copy.live.dir}/path/to/dir/*" escape="false" />
       <echo msg="Setting Write Permissions..." />
       <exec command="ssh -i /path/to/ssh/preshared/key remoteuser@remotehost chmod 777 ${copy.live.chmod}" escape="false" />
</target>
```


Here we just use ssh and our pre-shared key to execute chown to match owner and group to that of what the particular site uses (or apache:apache) and to 777 certain folders (like upload folders, etc.).



### Concluding Remarks



Using the rsync over ssh and one-line ssh techinques above you can do more complicated tasks like [DBDeploy](http://www.davedevelopment.co.uk/2008/04/14/how-to-simple-database-migrations-with-phing-and-dbdeploy/) on a live server that denies external access to its databases (PROTIP: rsync the db folder, build.properties, and a copy of the build script containing the migrate task to a folder outside of the publicly visible httpdocs and use the ssh trick to invoke Phing and the migrate task).

If you have your application set up appropriately you can modify the upload tasks to have rsync use the --delete flag to ensure perfect synchronization (files deleted locally are deleted remotely as well). A caveat is if you do not ignore generated content directories (like user uploaded images), those will be deleted as well.

And finally, even if you dislike Phing as a build environment, since I'm manually passing commands you could feasibly convert all my commands to a simple .sh script or apply them to any other build environment you desire.
