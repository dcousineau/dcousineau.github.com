---
author: admin
comments: true
date: 2009-01-06 17:12:56
layout: post
slug: zend-framework-module-init-script-controller-plugin
title: Zend Framework Module Init Script (Controller Plugin)
wordpress_id: 251
categories:
- Development
tags:
- Controller Plugin
- PHP
- Zend Framework
---

Well, it's been a while since I've done a PHP update hasn't it? Well alls well here in CStat, I have a quickie for you folks.

Recently at work I had the need to run a script before every single controller (namely to add a plugin folder to Dwoo) for a specific module that I did not desire for any other modules.

I could have subclassed all my controllers to extend a custom action controller that handled this in the `init()` method, however I'm lazy so I wrote a quick Zend Controller Plugin to handle this for me.

What it does is grabs the module name from the request object on routeShutdown (the routeShutdown happens after the route has been parsed and the request object has been generated, but before the action controller is instantiated and executed). From there it uses `Zend_Font_Controller::getInstance()->getModuleDirectory(...)` to grab the literal path to the module directory, tacks on our init file name (in our case "init.php"), and if it exists execute it in its own clean environment.

The init.php file has access to `$this`, where `$this` is an instance of the controller plugin (in particular we can do `$this->getRequest()` inside the init script).

The code is as follows:


```php
<?php
/**
 * This checks the current request module's directory for an initFile (defaults
 * to init.php) and runs it before the controller is loaded.
 * 
 * @copyright 2009 Daniel Cousineau
 * @license http://opensource.org/licenses/mit-license.php MIT License
 * @version 0.1.0
 */
class My_Controller_Plugin_ModuleInit extends Zend_Controller_Plugin_Abstract
{
	public static $initFileName = "init.php";
	
	/**
	 * @param Zend_Controller_Request_Abstract $request
	 * @return null
	 */
	public function routeShutdown(Zend_Controller_Request_Abstract $request)
	{
		$moduleName = $request->getModuleName();
		
		$moduleDirectory = Zend_Controller_Front::getInstance()->getModuleDirectory($moduleName);
		
		//Trim the paths and filenames to prevent any problems
		$initFile = rtrim($moduleDirectory,'/\') . '/' . ltrim(self::$initFileName,'/\');
		
		$this->runInitFile($initFile);
	}
	
	/**
	 * Run the file in its own cleaned scope
	 * 
	 * @param string $_initFile location of the input file
	 * @return null
	 */
	protected function runInitFile($_initFile)
	{
		if( file_exists($_initFile) )
			include_once $_initFile;
	}
}
```


As always all code I post on this blog is covered under the MIT license.
