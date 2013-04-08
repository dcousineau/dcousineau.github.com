---

comments: true
date: 2010-05-11 10:18:38
layout: post
slug: netbeans-code-completion-and-your-zend_view
title: Netbeans Code Completion and your Zend_View
wordpress_id: 472
categories:
- Development
tags:
- Netbeans
- Zend Framework
- Zend_View
---

Oh, look at me, blogging again! I definitely have a lot to blog about as I get the time, I'm coming off of a really involved project and learned a lot of tips I'd like to share about the Zend Framework. In the mean time I thought I'd share something I had a helluva time figuring out.

If you're using Zend Framework and NetBeans, you may be like me and bemoaning the lack of code completion in your Zend Views. As you may know, essentially what Zend_View does is includes your view within a method that belongs to a Zend_View object. This gives your view some nice variable encapsulation as well as access to the `$this` object (which is how Zend_View provides access to all the ViewHelpers and other functions).

Unfortunately NetBeans can't figures this out (such is the problem with static analysis on a dynamic language) without help. If you've been using NetBeans and its code completion you'll have already noticed that the PHPDocumentor syntax for `@var` or `@return` is how NetBeans figures out much of its code completion information, but that syntax doesn't work in a view script.

Thanks to [Mystic at tiplite.com](http://www.tiplite.com/useful-netbeans-6-8-php-tips/) I now know that to have code completion in your Zend_View scripts in NetBeans, add the following to the top of your view script:


```php
<?php
/* @var $this Zend_View */

```


Obviously if you're using custom Zend_View objects you can pass in their class name instead.
