---

comments: true
date: 2008-06-02 01:28:38
layout: post
slug: hierarchical-data-with-php-and-mysql
title: Hierarchical Data With PHP and MySQL
wordpress_id: 54
categories:
- Development
tags:
- Algorithms
- Hierarchical Data
- MySQL
- PHP
---

I recently had fun with an all-to-common issue with SQL driven websites: hierarchical data. For those who don't like big words, think trees. [Other people](http://dev.mysql.com/tech-resources/articles/hierarchical-data.html) have already discussed storage methods, and I would actually highly suggest you read the writeup if you haven't already.

While it is fairly straightforward to deal with, in our case we use [HTML_QuickForm](http://pear.php.net/package/HTML_QuickForm) to handle our forms and are using QuickForm's [hierselect](http://pear.php.net/manual/en/package.html.html-quickform.html-quickform-hierselect.setoptions.php) to select a category.

The issue starts showing its face in 2 distinct areas: (1) the client is not yet sure how deep they need their categories to go, and (2) the hierselect requires a very specific format of data to be passed in.

<!-- more -->

To start off we have our categories table:


```sql
CREATE TABLE  `categories` (
  `id` int(10) unsigned NOT NULL auto_increment,
  `parent_id` int(10) unsigned default NULL,
  `title` varchar(255) NOT NULL,
  `active` tinyint(1) NOT NULL default '1',
  PRIMARY KEY  (`id`),
  KEY `fk_categories_categories` (`parent_id`),
  CONSTRAINT `fk_categories_categories` FOREIGN KEY (`parent_id`) REFERENCES `categories` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

```


To make life easier for myself I have my model auto generating the SQL statement to go n levels deep in printing all categories:


```php
<?php
if( intval($depth) < 1 )
	$depth = 1;

$select = array();
$from = array();
$where = array();
$order = array();

for( $i = 1; $i <= $depth; $i++ )
{
	$select[] = "level" . $i . ".id AS level" . $i . "_id";
	$from[] = "`categories` AS level" . $i . "";
	$where[] = "( level" . $i . ".active = 1 OR level" . $i . ".active IS NULL )";
	$order[] = "level" . $i . "_id";
}

//SELECT
$sql = "SELECT " . implode(', ', $select) . ", level" . $depth . ".title AS title " ;

//FROM
$sql .= "FROM " . $from[0] . " ";

unset($from[0]);
if( count($select) > 0 )
{
	foreach( $from as $key => $value )
	{
		$from[$key] = $value . " ON level" . ($key) . ".id = level" . ($key+1) . ".parent_id";
	}
	
	$sql .= " RIGHT JOIN " . implode(" RIGHT JOIN ", $from) . " ";
}

//WHERE
$sql .= "WHERE level1.parent_id IS NULL AND " . implode(" AND ", $where) . " ";

//ORDER
$sql .= "ORDER BY " . implode(", ", $order);

$rs = $this->query($sql);

```


At a depth of 5 our SQL query generated becomes:


```sql

SELECT
  level1.id AS level1_id,
  level2.id AS level2_id,
  level3.id AS level3_id,
  level4.id AS level4_id,
  level5.id AS level5_id,
  level5.title AS title
FROM
  `categories` AS level1
  RIGHT JOIN `categories` AS level2 ON level1.id = level2.parent_id
  RIGHT JOIN `categories` AS level3 ON level2.id = level3.parent_id
  RIGHT JOIN `categories` AS level4 ON level3.id = level4.parent_id
  RIGHT JOIN `categories` AS level5 ON level4.id = level5.parent_id
WHERE
  level1.parent_id IS NULL
  AND ( level1.active = 1 OR level1.active IS NULL )
  AND ( level2.active = 1 OR level2.active IS NULL )
  AND ( level3.active = 1 OR level3.active IS NULL )
  AND ( level4.active = 1 OR level4.active IS NULL )
  AND ( level5.active = 1 OR level5.active IS NULL )
ORDER BY
  level1_id, level2_id, level3_id, level4_id, level5_id

```


The series of `( levelN.active = 1 OR levelN.active IS NULL )` guarantees that only entries with the active field turned on and that children of deactivated categories are not displayed. Fancy piece of SQL-fu if I do say so myself. The data is returned looking something like this:




level1_id level2_id level3_id level4_id level5_id title

1 Parent
2 Parent 2
1 7 Child 2
2 4 Child
2 5 Child 2
1 6 8 Grandchild
1 6 9 Grandchild 2
1 6 8 10 Grandchild
1 6 8 11 Grandchild 2
1 6 8 10 12 Great Grandchild
1 6 8 10 13 Great Grandchild 2


  

Now by cleaning up the null values from the $category array we have our path:


```php
<?php
$data = $rs->getRows();

$categories = array();
foreach( $data as $category )
{
	$title = $category['title'];
	unset($category['title']);
	
	foreach( array_reverse($category,true) as $key => $value )
	{
		if( $value === null )
		{
			unset($category[$key]);
		}
	}
	
	array_set($categories, array_merge(array(count($category)-1),array_values($category)), $title, "-- SELECT CATEGORY --");
}

return $categories;

```


You'll notice that we also insert into our `$categories` array according to the hierselect format linked above. That in and of itself is an interesting function as, for example, our entry for Great Great Grandchild would look like `$categories[4][1][6][8][10][12] = 'Great Great Grandchild';` and we need empty entries for each level.

`array_set()` has this code:


```php
<?php
function array_set(array &$array, array $keys, $value, $emptyText)
{
	$tmp =& $array;
	
	$lastKey = array_pop($keys);
	foreach( $keys as $key )
	{
		if( !is_array($tmp[$key]) )
		{
			$tmp[$key] = array();
		}
		
		$tmp =& $tmp[$key];
	}
	
	if( !isset($tmp['']) )
	{
		$tmp[''] = $emptyText;
	}
	
	$tmp[$lastKey] = $value;
}

```


And our final array output will look something like this:


```text
Array
(
    [0] => Array
        (
            [] => -- SELECT CATEGORY --
            [1] => Parent
            [2] => Parent 2
        )

    [1] => Array
        (
            [1] => Array
                (
                    [] => -- SELECT CATEGORY --
                    [6] => Child
                    [7] => Child 2
                )

            [2] => Array
                (
                    [] => -- SELECT CATEGORY --
                    [4] => Child
                    [5] => Child 2
                )
        )

    [2] => Array
        (
            [1] => Array
                (
                    [6] => Array
                        (
                            [] => -- SELECT CATEGORY --
                            [8] => Grandchild
                            [9] => Grandchild 2
                        )
                )
        )

    [3] => Array
        (
            [1] => Array
                (
                    [6] => Array
                        (
                            [8] => Array
                                (
                                    [] => -- SELECT CATEGORY --
                                    [10] => Grandchild
                                    [11] => Grandchild 2
                                )
                        )
                )
        )

    [4] => Array
        (
            [1] => Array
                (
                    [6] => Array
                        (
                            [8] => Array
                                (
                                    [10] => Array
                                        (
                                            [] => -- SELECT CATEGORY --
                                            [12] => Great Grandchild
                                            [13] => Great Grandchild 2
                                        )
                                )
                        )
                )
        )
)
```


Good thing is all that needs to be changed to dig deeper into the hierarchy is the `$depth` variable from above.

This solution is very good for write intensive applications, so anyone building a forum-like system will find use out of this system.

However I'm thinking about moving to the Nested Set model as the application for this code will not be write-intensive...
