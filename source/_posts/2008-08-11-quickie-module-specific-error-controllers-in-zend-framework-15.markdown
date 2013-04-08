---
author: admin
comments: true
date: 2008-08-11 11:12:44
layout: post
slug: quickie-module-specific-error-controllers-in-zend-framework-15
title: 'Quickie: Module-specific Error Controllers in Zend Framework (1.5)'
wordpress_id: 113
categories:
- Development
tags:
- Controller Plugin
- PHP
- Zend Framework
---

In my quest to do some alterations on ZF error handling (in particular, render the view if the action or controller is not found, makes it real easy for my designer to prototype) I had the desire to be able to allow modules to have their own ErrorControllers. Unfortunately, the Zend_Controller_Plugin_ErrorHandler() default does not allow for this and I didn't really want to extend that class (I planned on handling the rendering in the ErrorControllers) so I wrote up a quick plugin. Doing random stuff like this has really helped me get to know the Zend Framework (I was developing my own internal framework architectured somewhat similarly that I have abandoned due to time constraints and the large community behind ZF).

The plugin hooks into the routeShutdown as this is the first place I have access to the ZF-determined module name. I need the current module name to make sure I override the ErrorHandler's module, and while I could determine that for myself, problems would arise if I decided to start overriding the routers and URL structure.


```php
<?php
class MyApp_ControllerPlugin_ErrorControllerSelector extends Zend_Controller_Plugin_Abstract
{
	public function routeShutdown(Zend_Controller_Request_Abstract $request)
	{
		$front = Zend_Controller_Front::getInstance();
		
		//If the ErrorHandler plugin is not registered, bail out
		if( !($front->getPlugin('Zend_Controller_Plugin_ErrorHandler') instanceOf Zend_Controller_Plugin_ErrorHandler) )
			return;
		
		$error = $front->getPlugin('Zend_Controller_Plugin_ErrorHandler');
		
		//Generate a test request to use to determine if the error controller in our module exists
		$testRequest = new Zend_Controller_Request_HTTP();
		$testRequest->setModuleName($request->getModuleName())
		            ->setControllerName($error->getErrorHandlerController())
		            ->setActionName($error->getErrorHandlerAction());
		
		//Does the controller even exist?
		if( $front->getDispatcher()->isDispatchable($testRequest) )
		{
			$error->setErrorHandlerModule($request->getModuleName());
		}
	}
}

$front = Zend_Controller_Front::getInstance();
$front->registerPlugin( new MyApp_ControllerPlugin_ErrorControllerSelector() );
?>
```


You'll notice I generated a Zend_Controller_Request_HTTP() object and filled it with the ErrorController's set controller name and action name (again if I decided to change how the error controllers are named, this plug in will still work) then use the dispatcher to check if this controller exists (basically, can we dispatch this controller's action from this module). If so, go ahead and set the ErrorHandler's module name to the current module name.

I'll bring more later once I finish making the Zend Framework my bitch.
