---

comments: true
date: 2008-07-21 13:28:06
layout: post
slug: php-53-and-closures
title: PHP 5.3 and Closures
wordpress_id: 62
categories:
- Development
tags:
- Closures
- PHP
---

I'm fairly excited because as of only a week or so ago I found the [closures rfc](http://wiki.php.net/rfc/closures), and not only did I find that the rfc has a working patch, apparently it's already in PHP 5.3.

You see, apparently it's not good enough that we get name spaces and late static binding, oh no, we get closures too!

I downloaded the Win32 snapshot from [PHP snapshots page](http://snaps.php.net/) and indeed, closure support was included.


```php
$closure = function ($args) use ($global) { /*Body*/ };
```


According to the rfc, one must manually define external variables used within a function, however in my own tests you can still use the `global` keyword. The difference between the two is the `use` statement preserves the value of `$global` at creation and the `global` keyword would work as you would expect it to with a normal function. For example:


```php
<?php
$global = "Global Variable";

$closureUse = function ($arg) use ($global) { echo $global . " - " . $arg; };
$closureGlobal = function ($arg) { global $global; echo $global . " - " . $arg; };

echo "Basic Closure Testsn";
echo "-------------------nn";

echo "$closureUse('test'): ";
$closureUse('test');
echo "n";

echo "$closureGlobal('test'): ";
$closureGlobal('test');
echo "n";

echo "n";
echo "Global Closure Testsn";
echo "--------------------nn";



$global = "Global Variable (Changed)";



echo "$closureUse('test') (changed $global): ";
$closureUse('test');
echo "n";

echo "$closureGlobal('test') (changed $global): ";
$closureGlobal('test');
echo "n";
```


Would output:


```text
Basic Closure Tests
-------------------

$closureUse('test'): Global Variable - test
$closureGlobal('test'): Global Variable - test

Global Closure Tests
--------------------

$closureUse('test') (changed $global): Global Variable - test
$closureGlobal('test') (changed $global): Global Variable (Changed) - test
```


You can, as well, return references with a closure by putting the & between the function keyword and the parenthesis.


```php
$closureReturnsReference = function & ($args) use ($global) { /*Body*/ };
```


Just one of many shiny fancy things to look forward to in PHP 5.3.



### Edit 7/23/2008:



I should mention to the people that trashed my examples on reddit, I'm sorry for assuming you knew  what closures were and the the real world uses of them. I will try to refrain from giving a concise example of how a new language feature interacts with existing features and conventions, especially in PHP where things are a bit disorganized.
