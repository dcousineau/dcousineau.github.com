---

comments: true
date: 2009-10-20 17:32:40
layout: post
slug: doctrine-migrations-proper
title: Doctrine Migrations Proper
wordpress_id: 462
categories:
- Development
tags:
- zendcon09
---

I was talking with someone (I will edit this post when I find her card and remember her name) here at ZendCon and discovered that they were having trouble with migrations in Doctrine. Having gone through the same issues of Doctrine seemingly not being able to figure out your changes and generate migration classes, I thought I'd post the solution here for future reference.

Assuming you have access to the Doctrine CLI tool, the sequence you need is:
	
  1. Make the changes to your YAML schema files
  2. Run `$ doctrine generate-migrations-diff` to generate the migration deltas
  3. Run `$ doctrine migrate` to determine if your migration deltas are working (a concern if you're using SQL Server)
  4. Run `$ doctrine generate-models-yaml` to update your models to reflect the status of your database

The reasoning behind this, and if I'm wrong someone can correct me, is the "generate-migrations-diff" task compares the differences between your YAML schema files and your current **models**. If you have updated your models before generating the diff, obviously Doctrine will find no difference and generate no migration delta.
