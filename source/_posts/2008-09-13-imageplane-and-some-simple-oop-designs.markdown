---

comments: true
date: 2008-09-13 20:35:32
layout: post
slug: imageplane-and-some-simple-oop-designs
title: ImagePlane and some simple OOP designs
wordpress_id: 151
categories:
- Development
tags:
- Design Patterns
- OOP
- PHP
---

At [Net Perspective](http://www.net-perspective.com/) we created an image editor in flash that we just released for sale called [ImagePlane](http://kitchen.net-perspective.com/commercial/image-plane).

While ImagePlane has already been [creatively introduced](http://blog.net-perspective.com/2008/09/10/image-plane-as-easy-as/) and [documented quite thoroughly](http://kitchen.net-perspective.com/docs/image-plane/developer/), I wanted to go over some decisions concerning the PHP demo.

Currently ImagePlane posts saved data to URL via the HTTP POST method. Currently ImagePlane posts the entire Base64 encoded image (after transformations), but when designing the example class I had to account for the fact that I might have to someday support just a list of transformations (for the server to duplicate rather than receiving the entire image again). I also would have to support more than 1 output engine (despite the fact that I myself would just be using GD).

In designing my interface I like to make a quick list of operations and objects I will be expected to perform (no, this isn't a waterfall document, the list can be broad but I need a general idea of where I'm going):

  1. Set a filename (and by proximity, output directory and type)
  2. Select an output engine (either for performance reasons or simply support)
  3. Constrain image sizes
  4. Set certain image options (JPEG quality, possibly PNG transparency)

Now since I know I needed to support multiple output engines, I'm guaranteed to use the [Adapter pattern](http://en.wikipedia.org/wiki/Adapter_pattern). So now we have to split up our tasks among two classes.

Since our Adapter classes will each be very different internally (since they have to emulate and mimic features with different libraries) we'll want to keep them as small and clean as possible. Stepping through the list above we can see that 3 and 4 are obviously a best fit for our Adapter class since they deal directly with the actual image processing. All 2 really happens to be is a `setOutputEngine(Adapter_Interface $engine)` type method, however you can make it fancier by accepting a string for the engine and finding and instantiating the engine manually, but that's implementation side. Item 1 could be construed as belonging with the image processing but in reality it does not. Setting the output location and type is universal to all libraries, infact it really has nothing to do with image processing.

So we have a nice delegation of tasks to our objects we can start building. First things first, after creating our main shell class we create an interface (since things are so wildly different amongst image processing engines there's not much we can replicate so an interface will work better here as opposed to inheriting an abstract class) for our Adapter class.

The adapter class is going to need to do a few things. Specifically its going to need to load from a string (base64 encoded), set a few options (preserveTransparency and setOutQuality, for jpegs), and save the image data.


```php
<?php
interface ImagePlane_Output_Interface
{
	/**
	 * @param string $data base64 encoded image
	 * @return ImagePlane_Output_GD fluent interface
	 */
	public function loadFromString($image);
	
	/**
	 * @param bool $bool
	 * @return ImagePlane_Output_GD fluent interface
	 */
	public function preserveTransparency($bool = true);
	
	/**
	 * Only really used for JPEG
	 * 
	 * @param int $quality
	 * @return ImagePlane_Output_GD fluent interface
	 */
	public function setOutQuality($quality);
	
	/**
	 * @param string $outFile output location
	 * @param string $type Image type
	 * @return bool
	 */
	public function save($outFile, $type = ImagePlane::TYPE_JPEG);
}
```


In our primary class we are probably best splitting up the outfile into 3 separate segments. We should force good filenames by making the user put in the output directory, output name, THEN output type. This way we can easily fix our filenames, check directory problems, and enforce output type (and proper extensions) without having to hack through `pathinfo` results only to find the user specified some weird extension for a type that we didn't account for. Plus this leaves us open to in the future specifying multiple output files without having to include directories on all them, or specifying multiple output types again without having to retype the entire filename.


```php
<?php
class ImagePlane
{
	/**
	 * ImagePlane is passing in final transformed image via base64 POST
	 */
	const INPUT_POST = 'post';
	/**
	 * CURRENTLY NOT IMPLEMENTED BY IMAGEPLANE
	 */
	const INPUT_TRANSFORMATIONS = 'transformations';
	
	//Image Types
	const TYPE_JPEG = 'jpg';
	const TYPE_PNG = 'png';
	const TYPE_GIF = 'gif';
	
	/**
	 * Output engine
	 *
	 * @var ImagePlane_Output_Interface
	 */
	protected $outputEngine;
	
	/**
	 * Output directory
	 * 
	 * @var string
	 */
	protected $outDirectory = null;
	
	/**
	 * Output name (without extension)
	 *
	 * @var string
	 */
	protected $outName = null;
	
	/**
	 * Output type
	 *
	 * @var string
	 */
	protected $outType = null;
	
	/**
	 * Create new ImagePlane instance using a certain output engine
	 *
	 * @param ImagePlane_Output_Interface $outputEngine
	 */
	public function __construct(ImagePlane_Output_Interface $outputEngine)
	{
		$this->setOutputEngine($outputEngine);
	}
	
	/**
	 * Set the image output directory
	 * 
	 * @param string $directory
	 * @return ImagePlane fluent interface
	 */
	public function setOutDirectory($directory)
	{
		if( !is_writable($directory) )
		{
			throw new Exception("Output directory is not writeable");
		}
		
		$this->outDirectory = $directory;
		
		return $this;
	}
	
	/**
	 * Get the image output directory
	 * 
	 * @return string
	 */
	public function getOutDirectory()
	{
		return rtrim($this->outDirectory, '\/');
	}
	
	/**
	 * Output filename (without extension)
	 * 
	 * The filename is automatically stripped of any characters other than 'A-Z',
	 * '0-9', '_', and '-', and are replaced with a dash.
	 *
	 * @param string $name
	 * @return ImagePlane fluent interface
	 */
	public function setOutName($name)
	{
		$this->outName = preg_replace("/[^[:alnum:]_-]+/", '-', $name);
		
		return $this;
	}
	
	/**
	 * Get the output filename (without extension)
	 * 
	 * @return string
	 */
	public function getOutName()
	{
		return $this->outName;
	}
	
	/**
	 * Set output type.
	 * 
	 * Valid options are:
	 *  - ImagePlane::TYPE_JPEG
	 *  - ImagePlane::TYPE_PNG
	 *  - ImagePlane::TYPE_GIF
	 *
	 * @param string $type
	 * @return ImagePlane fluent interface
	 */
	public function setOutType($type)
	{
		if( !in_array($type, array(self::TYPE_PNG, self::TYPE_JPEG, self::TYPE_GIF)) )
			throw new Exception("Invalid Type");
		
		$this->outType = $type;
		
		return $this;
	}
	
	/**
	 * Get the output type
	 * 
	 * Valid returned types will be:
	 *  - ImagePlane::TYPE_JPEG
	 *  - ImagePlane::TYPE_PNG
	 *  - ImagePlane::TYPE_GIF
	 * 
	 * @return string
	 */
	public function getOutType()
	{
		return $this->outType;
	}
	
	/**
	 * Process ImagePlane input
	 *
	 * @param array $data
	 * @param string $type ImagePlane::INPUT_POST (default) or ImagePlane::INPUT_TRANSFORMATIONS
	 * @return bool
	 */
	public function process($data, $type = self::INPUT_POST)
	{
	
	}
	
	
	/**
	 * Set output engine
	 *
	 * @param ImagePlane_Output_Interface $outputEngine
	 */
	public function setOutputEngine(ImagePlane_Output_Interface $outputEngine)
	{
		$this->outputEngine = $outputEngine;
	}
	
	/**
	 * Get the output engine object
	 * 
	 * @return ImagePlane_Output_Interface
	 */
	public function getOutputEngine()
	{
		return $this->outputEngine;
	}
}
?>
```


You can see that on my set* functions I return the `$this` variable. This is called the fluent interface, and it allows you to make calls like `$object->method()->method()->method();` which makes for compact, and in some cases, semantic code.

You can see that spending just a little bit of time we now have an easily alterable utility class to process uploads from ImagePlane with a very low footprint for the actual end user:


```php
<?php
require_once '../php/imageplane.php';
spl_autoload_register(array('ImagePlane','autoload'));

$imagePlane = new ImagePlane(new ImagePlane_Output_GD());

$imagePlane->setOutDirectory(dirname(__FILE__))
           ->setOutName("IPOutput")
           ->setOutType(ImagePlane::TYPE_JPEG)
           ->process($_POST);
```


Now obviously this literal code snippet presents security issues but they can be tackled. Firstly the above code snippet can easily be moved inside an if block checking for authentication credentials (like an `$acl->isLoggedIn()` or similar), as well as checking for tokens via the `otherData` input variable as described in the [documentation](http://kitchen.net-perspective.com/docs/image-plane/developer/input#inputVariablesHeader).

Stay tuned, hopefully I'll be posting more in the future on OO design and provide more complex design topics in the future.

In the mean time if you want an excellent, classic book, I suggest you grab a copy of [Design Patterns: Elements of Reusable Object-Oriented Software](http://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional/dp/0201633612) by The Gang of Four.
