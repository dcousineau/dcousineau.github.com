---

comments: true
date: 2008-03-14 00:34:59
layout: post
slug: oo-design-and-php
title: OO Design and PHP
wordpress_id: 32
categories:
- Development
tags:
- Design Patterns
- OOP
- PHP
---

I was helping a coworker the other day with some object oriented design issues in PHP and thought it would be great to share with the general PHP community as the general PHP community seems to lack in good OOP skills.

The task, which will be detailed later when we release it, boiled down to having a collection of files we needed to perform an operation on. Sometimes the entire collection, sometimes just a subgroup. Currently the system will only support 1 kind of file however in the very near future a new file will be added. My coworker tackled it well but a few problems that I helped him out with. Namely he made a few common mistakes in the design that most PHP developers make, namely being inconsistent with naming/etc. and not utilizing Interfaces and argument type-hinting to enforce good design, as well as putting logic in the wrong classes.

One of the few pitfalls that did happened was placing the `render()` function in the collection class. While this would not be a problem for a single type, when extending the system to handle **n** node types we come across unnecessary complexity issues in adding to the `render()` function, as well as making it nigh-impossible to keep the original includes intact (so as to make upgrades work without the hassle of merging code).

<!-- more -->

First thing you should do is break down what you are wanting to do into _objects_ (not code... yet). A good way to do this is to describe what you wish to achieve in as few words as possible, for example "We have a _collection_ of _files_ to be processed". Notice two things: (1) we aren't including any subfeatuers that we know of ahead of time (like file groupings, rendering) and (2) two distinct objects rise forth. We have a _file_, a very literal, physical object; and we have a _collection_ of files, a more abstract concept. Already we can tell that concrete operations should probably stick with the files and abstract, reaching operations should obviously stick with the collection. Therefore lets begin with a Collection class and a Node class:


```php
class Collection
{
	public function __construct()
	{
	
	}
}

class Node
{
	public function __construct()
	{
	
	}
}
```


Now we move forward, lets expand the scope of the project a bit. Since we aren't describing individual functions just yet, our next feature to take into account is that there will be more than one type of file, but the operations will be the same. Any concept along the lines of performing the same operations on different objects probably will want an Interface. While in PHP we can get by without using an interface, it is good practice to use them as well as it makes the code easier to read, easier to understand, and easier to debug as we can use argument type-hinting to throw _accurate_ errors should an incorrect value be passed. I personally feel that this mandates the renaming of the Node class as our file objects will be of a node type (being of a NodeType type or NodeInterface type doesn't sound very natural). So now we have:


```php
interface Node
{
}

class TypeANode implements Node
{
	public function __construct()
	{
	
	}
}
```


Now we have the ability to add anywhere from 1 to n file type objects and use them in our system, so long as they implement our Node interface (to ensure mostly perfect compatibility).

Our next concept is now _rendering a file_. Some people might put the render function in the Collection class as we are rendering a file, the rendering action is external to the file as it takes a file, therefore it belongs outside of the Node-based class. **WRONG!** Our `render()` function belongs inside the _Node-based_ class BECAUSE it takes a (singular) file. Remember, a function inside of a class is called a method, render is a method we are performing on a file, therefore `render()` is a method for the Node-based class.

Finally we need to _add nodes_ to and _process_ our _collection_. Obviously we need such functions in our collection class. Now these should be straightforward to implement, however I should point out some new language features that allow you to make your code easier to maintain and debug. First there's argument type-hinting which makes life wonderful by throwing errors before anything is executed. For example:


```php
public function addNode( Node $n, $moduleName = 'default' )
```


Passing a variable not of type Node (e.g. not implementing our Node interface) will halt the application, preventing any accidental data loss during development or implementation. Finally, everyone should really get in the habit of using Exceptions. For those stalwart functional programming types, think of exceptions as a shortcut to using `trigger_error()`. For those of us object oriented programmers, I don't have to tell you about the benefits of being able to easily catch errors and perform damage control or alternative operations.

Hopefully as many PHP programmers who weren't or aren't object oriented as they should be (or don't understand all the hubub) as possible read this article. The PHP community as a whole needs to continue to push forward towards better programming practices, including shedding old, poorly written tutorials and replacing them with a proper, yet still easy to get started, education. In keeping inline with easy to start, below is a sample working implementation of the code discussed that you can try out for yourself:


```php
class Collection
{
	const ALL = -1;
	
	private $nodes = array();
	
	public function __construct()
	{
	
	}
	
	public function addNode( Node $n, $moduleName = 'default' )
	{
		if( is_string($moduleName) )
		{
			$this->nodes[$moduleName][] = $n;
		}
		else
		{
			throw new Exception("Argument 2 (module name) must be a string");
		}
	}
	
	public function render( $moduleName = 'default' )
	{
		$render = '';
		
		if( $moduleName === self::ALL )
		{
			foreach( array_keys($this->nodes) as $key )
			{
				$render .= "::In Module '" . $key ."'::n";
				$render .= $this->render( $key );
			}
		}
		else
		{
			if( isset($this->nodes[$moduleName]) )
			{
				foreach( $this->nodes[$moduleName] as $module )
				{
					$render .= $module->render() . "n";
				}
			}
		}
		
		return ($render !== '' ? $render : false);
	}
}

interface Node
{
	public function render();
}

class TypeANode implements Node
{
	private $name;
	
	public function __construct($name)
	{
		$this->name = (string)$name;
	}
	
	public function render()
	{
		return $this->name . " Rendered!";
	}
}
```


And some sample test code to try it all out:


```php
$sample = new Collection();

$sample->addNode( new TypeANode('test 1') );
$sample->addNode( new TypeANode('test 2'), 'module 1' );
$sample->addNode( new TypeANode('test 3'), 'module 2' );

var_dump($sample);

var_dump($sample->process());

var_dump($sample->process('module 1'));

var_dump($sample->process(Collection::ALL));
```


Your output (if you have [xdebug](http://xdebug.org/) installed it will be formatted like this as well) should look like:


```

**object**(_Collection_)[_1_]
  _private_ 'nodes' => 
    **array**
      'default' => 
        **array**

          0 => 
            **object**(_TypeANode_)[_2_]
              ...
      'module 1' => 
        **array**
          0 => 
            **object**(_TypeANode_)[_3_]
              ...
      'module 2' => 
        **array**

          0 => 
            **object**(_TypeANode_)[_4_]
              ...

```

```
string 'test 1 Rendered!
' _(length=17)_


```

```
string 'test 2 Rendered!
' _(length=17)_


```

```
string '::In Module 'default'::
test 1 Rendered!
::In Module 'module 1'::
test 2 Rendered!
::In Module 'module 2'::
test 3 Rendered!
' _(length=125)_


```

