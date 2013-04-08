---

comments: true
date: 2008-08-05 10:45:04
layout: post
slug: displaying-n-deep-trees-remember-your-algorithms-course
title: Displaying N-Deep Trees (Remember Your Algorithms Course?)
wordpress_id: 97
categories:
- Development
tags:
- Algorithms
- Hierarchical Data
- MySQL
- PHP
---

Often times when doing web development one has the need to categorize things. And not only basic categorization, n-deep hierarchical categorization. I've already discussed [storage and retrieval](http://www.toosweettobesour.com/2008/06/02/hierarchical-data-with-php-and-mysql/) of such data, but there comes a time when one needs to display this information.

Sometimes people build systems to only account for their current requirements. If the software calls for only 2 levels of categorization (Parent and Child only), a simple nested for loop will suffice. However, software requirements change and you'll soon find yourself up shit creek without a paddle if you need to support 3 or 4 levels of nesting.

For those of us who have formal computer science training, the answer comes rather quickly. To those who's training is less formal (most web developers I meet have practical training, not formal), I'll help you out: [Tree Traversals](http://en.wikipedia.org/wiki/Tree_traversal) (or if you are completely lost, [Recursion](http://en.wikipedia.org/wiki/Recursion)). If you know absolutely nothing about recursion, I would suggest you familiarize yourself now. A recent, easy to read example can be found [here](http://www.phpwomen.org/forum/index.php?t=msg&th=477).

<!-- more -->

To begin, lets outline the structure of our data. In my case, I am building a store. My designer requested our URLs to follow the convention /shop/PARENT_CATEGORY/[.../]CURRENT_CATEGORY/, where as the full path to our current category is in the URL and moving up the tree is as simple as popping a 'folder' off the URL. For instance, Bats are a child of Sports Equipment, so our URL would looks something like: /shop/sports-equipment/bats/. To view sports equipment, we just do /shop/sports-equipment/. What this means to me is I will always have the full path to the current leaf, which comes into play later.

Now my categories are returned to me in a nested associative array that contain the keys 'title', 'url', and 'children':


```text
**array**
  0 => 
    **array**
      'id' => string '1' _(length=1)_
      'title' => string 'Sports Equipment' _(length=14)_
      'url' => string 'sports-equipment' _(length=14)_
      'children' => 
        **array**
          0 => 
            **array**
              ...
          1 => 
            **array**
              ...
```


Obviously our ellipses continue the same pattern.

Since we want to print out our tree so that the root node is the first thing we see, the algorithm we desire is the **preorder tree traversal**. To put it simply, we work with the current node before running the recursive function call (A postorder tree traversal would run the recursive function call before working with the current node).

Now, with every recursive function you design your parameters around information that changes between each recursion, or information that needs to be passed along. In our case we'll obviously need to make our first parameter the categories array, we'll probably need to pass the path along, keep track of how deep we are, and because of our URL structure we need to keep track of what the path is at our current level in the tree.

So to begin, we have our shell function and call:


```php
function categories_display_recursive($categories, $path, $level = 0, $url_path = array())
{
	foreach($categories as $category)
	{
		//...
	}
}

echo '<ul>' . PHP_EOL;
categories_display_recursive($categories, $path);
echo '</ul>' . PHP_EOL;
```


Now some more requirements come into play. My designer has the menu set to default to a collapsed state and desires the currently viewed category and its parent(s) be opened. Since he also wants all the data on the page so the JavaScript does not have to waste time running AJAX requests, we'll just be adding an 'open' class to the nested ul and a 'selected' class to the li. Since I'm nice I decided to throw in a nice snippet to indent the code to make it easier for my designer to read: `implode('',array_fill(0, $level+1, "t"))`. Simply put, that line creates an array of size `$level+1` filled with tab characters, then implodes it down into a string.


```php
function categories_display_recursive($categories, $path, $level = 0, $url_path = array())
{
	foreach($categories as $category)
	{
		echo implode('',array_fill(0, $level+1, "t"));
		
		if( isset($path[$level]) && $path[$level] == $category['url'] )
		{
			echo '<li class="selected">';
		}
		else
		{
			echo '<li>';
		}
		
		echo '<a href="/shop/' . implode('/',array_merge($url_path,(array)$category['url'])) . '/">' . htmlentities($category['title']) . '</a>';
		
		if( count($category['children']) > 0 )
		{
			//...
		}
		
		echo '</li>' . PHP_EOL;
	}
}
```


And finally we need to descide what information the script needs to move forward. `$category['children']` is formatted exactly the same as `$categories` so we can pass that in and our recursive function won't know the difference. `$path` should be passed in unchanged, however ot make sure nothing is selected we should probably just pass in an empty array when printing children of an item that wasn't in our selected path. `$level` needs to be incremented and we need to add our current url (`$category['url']`) to the current iterations path (`array_merge()` works nicely here).

So, in the end we have:


```php
function categories_display_recursive($categories, $path, $level = 0, $url_path = array())
{
	foreach($categories as $category)
	{
		echo implode('',array_fill(0, $level+1, "t"));
		
		if( isset($path[$level]) && $path[$level] == $category['url'] )
		{
			echo '<li class="selected">';
		}
		else
		{
			echo '<li>';
		}
		
		echo '<a href="/shop/' . implode('/',array_merge($url_path,(array)$category['url'])) . '/">' . htmlentities($category['title']) . '</a>';
		
		if( count($category['children']) > 0 )
		{
			if( isset($path[$level]) && $path[$level] == $category['url'] )
			{
				echo PHP_EOL . implode('',array_fill(0, $level+1, "t"));
				echo '<ul class="open">' . PHP_EOL;
				
				categories_display_recursive($category['children'], $path, $level+1, array_merge($url_path, (array)$category['url']));
				
				echo implode('',array_fill(0, $level+1, "t"));
				echo '</ul>';
			}
			else
			{
				echo PHP_EOL . implode('',array_fill(0, $level+1, "t"));
				echo '<ul>' . PHP_EOL;
				
				categories_display_recursive($category['children'], array(),$level+1, array_merge($url_path, (array)$category['url']));
				
				echo implode('',array_fill(0, $level+1, "t"));
				echo '</ul>';
			}
		}
		
		echo '</li>' . PHP_EOL;
	}
}
```


Our final code should look exactly like this when Sports Equipment is the only element in the `$path` array:


```html
<ul id="nav">
	<li class="selected"><a href="/shop/sports-equipment/">Sports Equipment</a>
	<ul class="open">
		<li><a href="/shop/sports-equipment/bats/">Bats</a></li>
	</ul></li>
</ul>
```

