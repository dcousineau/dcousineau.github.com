---

comments: true
date: 2008-10-22 02:34:47
layout: post
slug: serialize-python-variables-to-php
title: Serialize Python Variables To PHP
wordpress_id: 202
categories:
- Development
tags:
- PHP
- Python
- Serialization
---

So I'm in the planning stages of a project where I'll probably be doing some heavy lifting in Python and serving up the output through PHP. Obviously this will entail transmitting data between Python and PHP and while I haven't had a chance to do performance testing to see if it's worth it to save to a database, the only other option was to serialize the data to a file that PHP could parse quickly. Obviously, the output from `serialize()` is going to be the fastest way to recieved the data.

While there already [exists](http://hurring.com/scott/code/python/serialize/) a Python set of classes that serializes (and serializes) Python data, it (a) didn't handle objects and (b) was licensed under the GPL (rant to follow later), meaning I couldn't use it for it's intended purpose due to the closed source nature of my project.

I decided this was a good enough time to play around with git so I created a github repo [here](http://github.com/dcousineau/phpserialize/tree/master).

It could probably still use some more TLC in the object area, maybe a few more supported types, and unserialization functions would be nice.

I do feel it wasn't bad for a few hours of hacking if I do say so myself.

The code for the module (should be in a phpserialize.py file) is as follows:

<!-- more -->



### Python Code




```python
"""
Collection of functions to serialize Python variables to the native PHP format

@version 0.1
@author Daniel Cousineau <dcousineau@gmail.com>
@copyright Copyright (c) 2008 Daniel Cousineau
@license http://www.opensource.org/licenses/mit-license.php MIT License
"""

from types import *

def serialize(var):
	"""
	Serialize var to the native PHP serialization format
	"""
	if type(var) is IntType or type(var) is LongType:
		return serialize_int(var)
	
	elif type(var) is FloatType:
		return serialize_decimal(var)
	
	elif type(var) is BooleanType:
		return serialize_boolean(var)
	
	elif type(var) is StringType:
		return serialize_string(var)
	
	elif type(var) is NoneType:
		return serialize_null()
	
	elif type(var) is ListType or type(var) is TupleType:
		return serialize_array(var)
	
	elif type(var) is DictType:
		return serialize_dictionary(var)
	
	elif type(var) is InstanceType:
		return serialize_object(var)
	
	else:
		raise TypeError("Invalid Type %s" % (type(var)))

def serialize_string(string):
	"""
	Format: s:L:"STRING";
		L: Length of the string
		STRING: The string itself. No need to escape quotes (")
	"""
	return "s:%d:"%s";" % (len(string), string)

def serialize_int(int):
	"""
	Format: i:D;
		D: the integer
	"""
	return "i:%d;" % (int)

def serialize_decimal(decimal):
	"""
	Format: d:D;
		D: the decimal (accepts expontent notation)
	"""
	return "d:%s;" % (decimal)

def serialize_boolean(boolean):
	"""
	Format: b:B;
		B: 1 for True, 0 for False
	"""
	return "b:%d;" % (boolean)

def serialize_null():
	"""
	Format: N;
	"""
	return "N;"

def serialize_array(array):
	"""
	Format: a:K:{VALUES}
		K: Number of entries (number of keys for dictionaries)
		VALUES: serialized key appended to serialized value. Do not worry about ;
		        if a value is an array
	
	DOES NOT END WITH A ;
	"""
	values = []
	for index, value in enumerate(array):
		values.append(serialize_array_key(index))
		values.append(serialize(value))
	return "a:%d:{%s}" % (len(array), "".join(values))

def serialize_array_key(var):
	"""
	Decimals and Boolans are automatically converted to integers for key values.
	Arrays and Objects throw errors if they are set as a key
	"""
	if type(var) is IntType or type(var) is FloatType or type(var) is BooleanType:
		return serialize_int(int(var))
	
	elif type(var) is NoneType:
		return serialize_int(0)
	
	elif type(var) is StringType:
		return serialize_string(var)
	
	else:
		raise TypeError("Invalid Key Type %s" % (type(var)))

def serialize_dictionary(array):
	"""
	See serialize_array()
	"""
	values = []
	for index, value in array.iteritems():
		values.append(serialize_array_key(index))
		values.append(serialize(value))
	return "a:%d:{%s}" % (len(array), "".join(values))

def serialize_object(obj):
	"""
	Format: O:L:"CLASS":K:{MEMBERS}
		L: Length of class name
		CLASS: class name (string)
		K: Number of class members (ignore 'static' members)
		MEMBERS: Members as an associative array (dictionary) (ignore 'static' members)
	"""
	objClass = obj.__class__.__name__
	
	values = []
	for index, value in obj.__dict__.iteritems():
		values.append(serialize_array_key(index))
		values.append(serialize(value))
	
	return "O:%d:"%s":%d:{%s}" % (len(objClass), objClass, len(obj.__dict__), "".join(values))

```


I also wrote up some tests in PHP:



### PHP Test Code




```php
<?php
echo "BEGINNING TESTS:n";
echo "----------------nn";

//STRING

$test = "String";
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize("String")
HEREDOC;

print_results("STRINGS", $test, $python);

//INTEGER

$test = 52;
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize(52)
HEREDOC;

print_results("INTEGERS", $test, $python);

//DECIMAL

$test = 9.8;
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize(9.8)
HEREDOC;

print_results("DECIMALS", $test, $python);

$test = 1.52*pow(10,-5);
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize(1.52*pow(10,-5))
HEREDOC;

print_results("DECIMALS (exponential format)", $test, $python);

//BOOLEAN

$test = true;
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize(True)
HEREDOC;

print_results("BOOLEANS", $test, $python);

//NULL

$test = null;
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize(None)
HEREDOC;

print_results("NULLS", $test, $python);

//ARRAY

$test = array(9, 8, 7);
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize((9, 8, 7))
HEREDOC;

print_results("SIMPLE ARRAYS", $test, $python);

$test = array("key" => "value", 5 => "another value");
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize({"key":"value",5:"another value"})
HEREDOC;

print_results("ASSOCIATIVE ARRAYS", $test, $python);

$test = array("key" => "value", 5 => array("values"));
$python = <<<HEREDOC
import phpserialize
print phpserialize.serialize({"key":"value",5:["values"]})
HEREDOC;

print_results("COMPLEX ARRAYS", $test, $python);

//OBJECTS

class Test
{
	public $i = "PUBLIC";
	public $j = "B0RKED";
}

$test = new Test();
$python = <<<HEREDOC
import phpserialize

class Test:
	def __init__(self):
		self.i = "PUBLIC"
		self.j = "B0RKED"

print phpserialize.serialize(Test())
HEREDOC;

print_results("OBJECTS", $test, $python);



function print_results($test_name, $var, $python_code)
{
	echo sprintf( "TESTING %s:nn", $test_name );

	echo sprintf( "Expecting:  %sn", serialize($var) );

	echo sprintf( "            %sn", implode("n            ", explode("n", print_r($var, true))) );

	echo "n";

	$recieved = exec("python -c " . escapeshellarg($python_code));

	echo sprintf( "Recieved:   %sn", $recieved );

	echo sprintf( "            %sn", implode("n            ", explode("n", print_r(unserialize($recieved), true))) );
	
	echo "n---nn";
}
```


Which gives us the following output:



### PHP Test Results




```text

BEGINNING TESTS:
----------------

TESTING STRINGS:

Expecting:  s:6:"String";
            String

Recieved:   s:6:"String";
            String

---

TESTING INTEGERS:

Expecting:  i:52;
            52

Recieved:   i:52;
            52

---

TESTING DECIMALS:

Expecting:  d:9.800000000000000710542735760100185871124267578125;
            9.8

Recieved:   d:9.8;
            9.8

---

TESTING DECIMALS (exponential format):

Expecting:  d:1.5200000000000001853270141516372149226299370639026165008544921875E-5;
            1.52E-5

Recieved:   d:1.52e-05;
            1.52E-5

---

TESTING BOOLEANS:

Expecting:  b:1;
            1

Recieved:   b:1;
            1

---

TESTING NULLS:

Expecting:  N;
            

Recieved:   N;
            

---

TESTING SIMPLE ARRAYS:

Expecting:  a:3:{i:0;i:9;i:1;i:8;i:2;i:7;}
            Array
            (
                [0] => 9
                [1] => 8
                [2] => 7
            )
            

Recieved:   a:3:{i:0;i:9;i:1;i:8;i:2;i:7;}
            Array
            (
                [0] => 9
                [1] => 8
                [2] => 7
            )
            

---

TESTING ASSOCIATIVE ARRAYS:

Expecting:  a:2:{s:3:"key";s:5:"value";i:5;s:13:"another value";}
            Array
            (
                [key] => value
                [5] => another value
            )
            

Recieved:   a:2:{i:5;s:13:"another value";s:3:"key";s:5:"value";}
            Array
            (
                [5] => another value
                [key] => value
            )
            

---

TESTING COMPLEX ARRAYS:

Expecting:  a:2:{s:3:"key";s:5:"value";i:5;a:1:{i:0;s:6:"values";}}
            Array
            (
                [key] => value
                [5] => Array
                    (
                        [0] => values
                    )
            
            )
            

Recieved:   a:2:{i:5;a:1:{i:0;s:6:"values";}s:3:"key";s:5:"value";}
            Array
            (
                [5] => Array
                    (
                        [0] => values
                    )
            
                [key] => value
            )
            

---

TESTING OBJECTS:

Expecting:  O:4:"Test":2:{s:1:"i";s:6:"PUBLIC";s:1:"j";s:6:"B0RKED";}
            Test Object
            (
                [i] => PUBLIC
                [j] => B0RKED
            )
            

Recieved:   O:4:"Test":2:{s:1:"i";s:6:"PUBLIC";s:1:"j";s:6:"B0RKED";}
            Test Object
            (
                [i] => PUBLIC
                [j] => B0RKED
            )
            

---

```

