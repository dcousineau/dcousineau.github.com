---

comments: true
date: 2008-05-15 14:08:21
layout: post
slug: automating-the-development-workflow
title: Automating the Development Workflow
wordpress_id: 48
categories:
- Development
tags:
- Automation
- DBDeploy
- Deployment
- Phing
- Project Management
- Xinc
---

I just rolled out some new automation tools for a few projects here at work and so far I've been extremely happy.

Much to my embarrassment, development has previously been outside of source control due to the fact that we develop sites, we don't deploy packaged applications, and we don't have a cohesive IT setup (everyone sets up their desktop to their liking so maintaining consistent development environments across all computers is difficult).

However, thanks to SVN, [Xinc](http://code.google.com/p/xinc/) and [Phing](http://phing.info/) (and [DBDeploy](http://dbdeploy.com/)), this has changed! Now everything is in source control and automatically deployed to our dev server upon commit. I am currently talking with Arno about perfecting svn tag monitoring to automate staging and (possibly) live deployments, so I'll post about what I did when that's finished.

The great thing about this setup is all pieces are technically interchangeable. If you don't like Xinc you can use [CruiseControl](http://cruisecontrol.sourceforge.net/). If you don't like Phing you can use [Pake](http://www.pake-project.com/), or a shell script even. If you don't like DBDeploy you can roll your own setup or swap it out for your database versioning system of your choice!

However this post will cover Xinc, Phing, and DBDeploy as (a) I have experience with them and none of the others, and (b) they integrate extremely well (Xinc and Phing's primary distribution method of choice are [PEAR](http://pear.php.net/) channels).

<!-- more -->

To get started, we'll of course need to install Phing and Xinc. **NOTE:** DBDeploy is ported to PHP and included with Phing, don't get your panties in a bunch trying to get the .jar files from DBDeploy's website to run because you don't need them!

First get Phing installed. As a forewarning, I like to use the --alldeps flag, however even if you don't I would suggest using --alldeps on Phing as most of the dependencies are good packages like PHPUnit, PHPdoc, and in particular the VersionControl_SVN, which is used by Xinc as well.

```text
# pear channel-discover pear.phing.info
# pear config-set preferred_state beta
# pear install --alldeps phing/Phing
```

During the install you will get a message about VersionControl_SVN being in alpha and will give you channel://... address. You can either copy that and run `pear install --alldeps channel://...` or download the tgz manually from [its PEAR page](http://pear.php.net/package/VersionControl_SVN/download).

Now we'll install Xinc. Xinc 2.0.1 recently has made use of the Graph component from [eZ components](http://ezcomponents.org/) so we'll need to add both Xinc and eZ Component's channel to PEAR before we install:

```text
# pear channel-discover pear.xinc.eu
# pear channel-discover components.ez.no
# pear install --alldeps xinc/Xinc
# pear run-scripts xinc/Xinc
```

The last line runs Xinc's install scripts (as Xinc installers a daemon into /etc/init.d/ as well as sets up a few folders outside of the usual PEAR directory). I would suggest sticking with the defaults, installing the sample project is not really necessary but I would suggest for newbies it as it is a full featured example you can use as a template.

Next you'll need to setup the Xinc display page. If you installed to the default locations you'll need to go to `/etc/xinc/`. In there is a `www.conf` file. You can just `ln -s` it to your conf.d folder in apache (in Ubuntu/Debian: `/etc/apache2/sites-available/`, in Fedora/Redhat: `/etc/httpd/conf.d/`). You will probably need to alter the VirtualHost tag to fit your server setup if you want external access, but it should work just fine out of the box.

If you used the default setup you can visit localhost:8080 to test and ensure it is up and running. You can visit [xinc.eu](http://www.xinc.eu/) to view a working demo (obviously well past the "Self Hosting" milestone).

From there we can begin to add our projects. I `chmod 777`'d and symlinked the `/etc/xinc/conf.d/` folder into the public share to provide access to the project xml files to the other developers in the office. For now lets leave it alone, don't worry about starting up the Xinc daemon as we need to setup our project first.



> The rest of the post will involve configuration of Phing and Xinc. This post will not go into detail on the configurations, for more information check the user documentation on their respective websites.



Now it's time to setup our Phing buildfile and project. Obviously you should adjust this setup to suit your project. We initially start with this directory/file layout:
	
* `db/`
  * `deltas/`
* `lib/`
* `reports/`
* `build.properties`
* `build.xml`


Lets start with build.xml. When you run `phing` in the command line without specifying the build file, it looks for a file named build.xml by default. Lets see an example build.xml file:

> The DBDeploy code comes with much thanks to Dave Marshall's [post](http://www.davedevelopment.co.uk/2008/04/14/how-to-simple-database-migrations-with-phing-and-dbdeploy/)

```xml
<!--?xml version="1.0" encoding="UTF-8"?-->
<project name="Brazos Valley API" default="dist" basedir=".">
	
	<tstamp>
	
	<property file="./build.properties">
	
	<fileset dir="${project.basedir}/lib" id="codefiles">
		<include name="**">
		<exclude name="**/.svn/**">
	</exclude></include></fileset>

	<target name="migrate" description="Database Migrations" depends="doc">
		<!-- delete leftover SQL scripts -->
		<delete>
			<fileset dir="${project.basedir}/db">
				<include name="*.sql">
			</include></fileset>
		</delete>
		
		<!-- load the dbdeploy task -->  
		<taskdef name="dbdeploy" classname="phing.tasks.ext.dbdeploy.DbDeployTask">  
		
		<!-- these two filenames will contain the generated SQL to do the deploy and roll it back-->  
		<property name="build.dbdeploy.deployfile" value="db/deploy-${DSTAMP}${TSTAMP}.sql">  
		<property name="build.dbdeploy.undofile" value="db/undo-${DSTAMP}${TSTAMP}.sql">  
		
		<!-- generate the deployment scripts -->  
		<dbdeploy url="mysql:host=${db.local.host};dbname=${db.local.name}" userid="${db.local.user}" password="${db.local.pass}" dir="${project.basedir}/db/deltas" outputfile="${project.basedir}/${build.dbdeploy.deployfile}" undooutputfile="${project.basedir}/${build.dbdeploy.undofile}">
		
		<!-- execute the SQL - Use mysql command line to avoid trouble with large files or many statements and PDO -->  
		<exec command="${progs.mysql} -h${db.local.host} -u${db.local.user} -p${db.local.pass} ${db.local.name} < ${build.dbdeploy.deployfile}" dir="${project.basedir}" checkreturn="true">
		
		<!-- prepare SQL file for artifact publishing -->
		<move file="${project.basedir}/${build.dbdeploy.deployfile}" tofile="${project.basedir}/db/deploy.sql" overwrite="true">
		<move file="${project.basedir}/${build.dbdeploy.undofile}" tofile="${project.basedir}/db/undo.sql" overwrite="true">
		
	</move></move></exec></dbdeploy></property></property></taskdef></target>
	
	<target name="copy" description="Copy Files To httpdocs">
		<exec command="rsync -Crd ${project.basedir}/lib/ ${copy.dir}" escape="false">
		<exec command="chown -R apache:apache ${copy.dir}" escape="false">
	</exec></exec></target>
	
	<target name="dist" depends="copy, migrate">
		
	</target>
</project>
```


The contents of build.properties:


```ini
# Property files contain key/value pairs
#key=value

copy.dir=/var/www/

# Credentials for the database migrations
db.local.host=
db.local.user=
db.local.pass=
db.local.name=

# Location of the MySQL program. DO NOT ALTER
progs.mysql=/usr/bin/mysql

```


The last portion of the file, the db.* and progs.mysql key-value pairs are used by db deply and should connect to the same database as your application.

You will need to run a simple query to add a table into your database before you can run the migrate task, however, since DBDeploy tracks its versioning information via a table in the database in question (in MYSQL dialect):


```sql
CREATE TABLE changelog (
  change_number BIGINT NOT NULL,
  delta_set VARCHAR(10) NOT NULL,
  start_dt TIMESTAMP NOT NULL,
  complete_dt TIMESTAMP NULL,
  applied_by VARCHAR(100) NOT NULL,
  description VARCHAR(500) NOT NULL
);

ALTER TABLE changelog ADD CONSTRAINT Pkchangelog PRIMARY KEY (change_number, delta_set);
```


Once you run the query above and supplied your database credentials we need to create our initial DB delta. DBDeploy deltas take on the naming convention [VERSION_NUMBER]-[COMMENT].sql. Comments are largely ignored and all the system looks for is the version number. So our first DBDeploy delta will be called something like `db/deltas/1-create_initial_schema.sql`.

The delta file will contain 2 SQL Query sets. The first is to perform whatever actions are necessary to bring the database up to revision n and the second set is the inverse: the queries necessary to remove revision n and restore the database to the previous state.

An example delta file that starts a users table:


```sql
--//

CREATE TABLE IF NOT EXISTS `users` (
  `id` int(10) unsigned NOT NULL auto_increment,
  `handle` varchar(25) NOT NULL default '',
  PRIMARY KEY  (`id`),
  UNIQUE KEY `users_handle_index` (`handle`)
) ENGINE=InnoDB  DEFAULT;

--//@UNDO

-- You will obviously want to ALTER TABLE ... DROP KEY to remove
-- any FK constraints at the very beginning if you are dropping tables

DROP TABLE IF EXISTS `users`;

--//
```


DBDeploy works by determining the most recent revision of the database (if changelog is empty then DBDeploy will use all available revisions in deltas), then marges every script between the current revision and the latest in deltas in order and saves the sql files. The `<tstamp/>` tag sets the timestamp variables (`${DSTAMP}${TSTAMP}` used to name the SQL files to ensure, even though we clear out old SQL files, the output will never overwrite an existing file. We late move them into generic undo.sql and deploy.sql files for later use.

Once you are done, make sure you commit the project to your SVN repository.

Now we need to setup Xinc. First things first, create your PROJECTNAME.xml file in `/etc/xinc/conf.d/`.

Lets start with this basic configuration:


```xml
<!--?xml version="1.0"?-->
<xinc engine="Sunrise">
	<project name="PROJECTNAME">

		<schedule interval="15">

		<property name="dir" value="/var/xinc/projects/PROJECTNAME">

		<modificationset>
			<svn directory="${dir}" update="true">
		</svn></modificationset>

		<builders>
			<phingbuilder buildfile="${dir}/build.xml" workingdir="${dir}" target="dist">
		</phingbuilder></builders>

		<publishers>
			<artifactspublisher file="${dir}/db/deploy.sql">
			<artifactspublisher file="${dir}/db/undo.sql">
		</artifactspublisher></artifactspublisher></publishers>
	</project>
</xinc>
```


Whenever Xinc runs a build, it stores all log output and any selected artifacts for each attempted build (even the failures, **especially** the failed builds). So if there are any files you would take a particular interest in the contents of during a build, it is advisable to publish it as an artifact! If you read through the Xinc documentation there are special publishers for documentation and unit test reports. Obviously, in our case we are publishing the deploy.sql and undo.sql files.

Once you have saved the PROJECTNAME.xml file we need to do one last setup before we can run the daemon. We need to navigate to `/var/xinc/projects/` and create a directory with the same exact name as the `name=""` property in the `project` tag of our PROJECTNAME.xml file (in this case, PROJECTNAME). In this directory you need to check out a copy of your project, `svn co http://REPO/LOCATION/trunk`.

Now that we've done all this we can do a `sudo /etc/init.d/xinc start` and Xinc should start right up!

If you find that the project does not appear on the web interface, this usually means it's corresponding folder in `/var/xinc/status/` has not been created and an easy fix is to commit some change and force Xinc into a build (Xinc usually isn't building because obviously at this point in time the checked out copy and the remote repository are perfectly in sync).

From here on out it should be easy to setup new projects if you keep around both an Xinc project.xml and SVN project template.
