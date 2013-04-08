---
author: admin
comments: true
date: 2008-10-10 10:42:13
layout: post
slug: outputting-csv-as-a-downloadable-file-in-php
title: Outputting CSV as a Downloadable File in PHP
wordpress_id: 193
categories:
- Development
tags:
- CSV
- PHP
---

Nearly every application you could write in for the business sphere in PHP probably requires some sort of data export, most likely in the CSV format.

The easiest way to provide a downloadable file is by altering the headers and echo'ing the file content. In our case:


```php
header("Content-type: text/csv");
header("Cache-Control: no-store, no-cache");
header('Content-Disposition: attachment; filename="filename.csv"');
```


We want to set our applicable Content-Type so that the browsers associate the file properly. Just relying on the extension doesn't work, even in Windows. The magic is in the third header setting, "Content-Disposition," which informs the browser to download as a separate file (don't open a new window and display a blank page, just display the file download box) and tell the browser the filename is "filename.csv". This way rewrite rules like http://localhost/export/csv/ will result in a download box that declares the file "filename.csv" rather than a randomly assigned name or whatever the current url is.

Into the meat of the CSV export. At the very beginning we need to open up a stream to the PHP output (the same place where echo sends its string content, which is NOT stdout):


```php
$outstream = fopen("php://output",'w');
```


Next we're going to assume you already have your data packed nicely into an array (or array of arrays) so long as we have a single array per row/line.

The magic comes into play using the build in PHP function [`fgetcsv()`](http://php.net/fgetcsv). `fgetcsv()` takes an array for a single row and outputs it, automatically escaping output according to column and enclosure delimiters!

`fgetcsv()` requires a file resource as its first parameter and the magic of PHP streams is they act like a file resource (actually a file resource is just a file stream), so we give it `$outstream` to make `fputcsv()` echo its output. We fill in the rest of the parameters according to the php.net documentation and voila we have:


```php
header("Content-type: text/csv");
header("Cache-Control: no-store, no-cache");
header('Content-Disposition: attachment; filename="filename.csv"');

$outstream = fopen("php://output",'w');

$test_data = array(
	array( 'Cell 1,A', 'Cell 1,B' ),
	array( 'Cell 2,A', 'Cell 2,B' )
);

foreach( $test_data as $row )
{
	fputcsv($outstream, $row, ',', '"');
}

fclose($outstream);
```


For more output stream craziness, [Timothy Boronczyk](http://zaemis.blogspot.com/) from the #phpc IRC channel on freenode shared a code snipped that outputs CSV either to the output buffer OR will return it as a string using some clever streams hackery:


```php
function exportCSV($data, $col_headers = array(), $return_string = false)
{
    $stream = ($return_string) ? fopen ('php://temp/maxmemory', 'w+') : fopen ('php://output', 'w');

    if (!empty($col_headers))
    {
        fputcsv($stream, $col_headers);
    }

    foreach ($data as $record)
    {
        fputcsv($stream, $record);
    }

    if ($return_string)
    {
        rewind($stream);
        $retVal = stream_get_contents($stream);
        fclose($stream);
        return $retVal;
    }
    else
    {
        fclose($stream);
    }
}
```

