---

comments: true
date: 2010-09-16 16:11:47
layout: post
slug: doctrine-1-2-mssql-alternative-limitpaging
title: Doctrine 1.2 MSSQL Alternative LIMIT/Paging
wordpress_id: 477
categories:
- Development
tags:
- doctrine
- mssql
---

At work I had been having all sorts of issues with `Doctrine_Connection_Mssql`'s `LIMIT` alteration, based on `Zend_Db`'s code.

The code used the more-compatible-with-SQL-Server-2000 technique of modifying the query to `SELECT TOP (offset + limit)`, reverse the `ORDER BY` clause and `SELECT TOP (limit)`, then finally reversing the returned dataset.

As ugly as this technique is, it works. The problem is it requires an extreme amount of intelligence or an extreme amount of simplicity in the query in order for an automated system like Doctrine to be usable. The biggest caveat with this technique is good goddamned luck paging your query if it doesn't have an `ORDER BY`. And sometimes queries that are complex enough break the modified `Zend_Db` code.

There exists an [easier MSSQL paging technique](http://varjabedian.net/archive/2008/04/09/paging-made-easy-in-ms-sql-server-2005.aspx). Using features first available in SQL Server 2005, with only 1 subquery you can mimic MySQL's `LIMIT` clause with ease.

Basically, Microsoft provided the following special feature to determine the row number in your final resultset:


```sql
SELECT Row_Number() OVER (ORDER BY column) AS RowIndex FROM table
```

Notice the `OVER (ORDER BY column)` segment? This is provided as this query will most often be used in a subquery. Given that MSSQL does not allow `ORDER BY` statements in subqueries, just move them into the `OVER (...)` section.

To borrow Ralph Varjabedian's example, the following MySQL query:

```sql
SELECT * FROM users LIMIT 15, 15
```

Becomes functionally equivalent to the following MSSQL query:


```sql
SELECT 
    * 
FROM 
    (
    SELECT 
        Row_Number() OVER (ORDER BY userID) AS RowIndex, 
        * 
    FROM 
        users
    ) AS sub 
WHERE 
    sub.RowIndex > 15 
    AND sub.RowIndex <= 30
```


Now all I needed to do was add the ability for Doctrine to parse a generated MSSQL query and reform it like the one above!

I've provided a copy of the Doctrine connection adapter I wrote. Simply add the following line to wherever you setup Doctrine:

**Please note: This was the result of about 6 hours of hacking today. There are certainly places in the code where it can be more robust or improved, especially in my little parser. Use at your own risk (though I have yet to encounter any errors in my application).**


```php
<?php
class CaseMan_Doctrine_Connection_Mssql extends Doctrine_Connection_Mssql
{
    /**
     * @var string $driverName                  the name of this connection driver
     */
    protected $driverName = 'Mssql';

    /**
     *
     * @var boolean $is2005OrBetter             cached result determining if server is SQL Server 2005 or better
     */
    protected $is2005OrBetter;

    /**
     * the constructor
     *
     * @param Doctrine_Manager $manager
     * @param PDO $pdo                          database handle
     */
    public function __construct(Doctrine_Manager $manager, $adapter)
    {
        parent::__construct($manager, $adapter);
    }

    /**
     * Adds an adapter-specific LIMIT clause to the SELECT statement.
     *
     * @param string $query
     * @param mixed $limit
     * @param mixed $offset
     * @return string
     */
    public function modifyLimitQuery($query, $limit = false, $offset = false, $isManip = false)
    {
        if ($limit <= 0 && $offset <= 0)
            return $query;
        
        if( !$this->is2005OrBetter() ) //Not at least 2005
            return parent::modifyLimitQuery ($query, $limit, $offset, $isManip);
        
        //SETUP FIELD ALIASES
        $inner_query_name = '_inner_query_';
        $row_count_name = '_inner_query_row_count_';

        //PREPARE TOKENIZER REGEXES
        $escaped = "\\\.";
        $sections = "SELECT|FROM|WHERE|GROUP[ ]+BY|HAVING|ORDER[ ]+BY";
        $open_delimiters = "[\[\(]";
        $close_delimiters = "[\]\)]";
        $string_delimiters = "[\"\']";

        //TOKENIZE QUERY
        $_split_query = preg_split(
            "#({$escaped}|{$sections}|{$open_delimiters}|{$close_delimiters}|{$string_delimiters})#i",
            trim($query),
            -1,
            PREG_SPLIT_DELIM_CAPTURE | PREG_SPLIT_NO_EMPTY
        );

        $query_parts = array();
        $_state = 'BEGIN';
        $_current_part = 'SELECT';
        $_stack = array();
        while( $_token = array_shift($_split_query) )
        {
            switch( $_state )
            {
                case 'BEGIN':

                    if( trim(strtoupper($_token)) == 'SELECT' )
                    {
                        $query_parts['SELECT'] = '';
                        $_current_part = 'SELECT';
                        $_state = 'SECTION';
                    }
                    else
                    {
                        throw new Doctrine_Exception("Invalid query passed to modifyLimitQuery, must begin with SELECT");
                    }

                    break;
                case 'SECTION':

                    if( preg_match("#^({$sections})$#i", trim($_token)) )
                    {
                        $_section = strtoupper(trim($_token));

                        $query_parts[$_section] = '';
                        $_current_part = $_section;
                        $_state = 'SECTION';
                    }
                    else
                    {
                        $query_parts[$_current_part] .= $_token;

                        if( preg_match("#^({$string_delimiters})$#i", trim($_token)) )
                        {
                            array_push($_stack, trim($_token));
                            $_state = 'STRING';
                        } else if( preg_match("#^({$open_delimiters})$#i", trim($_token)) )
                        {
                            array_push($_stack, trim($_token));
                            $_state = 'DELIMITED';
                        }
                    }

                    break;
                case 'DELIMITED':

                    $query_parts[$_current_part] .= $_token;

                    if( preg_match("#^({$close_delimiters})$#i", trim($_token)) )
                    {
                        $_prev_delimiter = array_pop($_stack);
                        switch( trim($_token) )
                        {
                            case ']':
                                if( $_prev_delimiter != '[' )
                                    throw new Doctrine_Exception("Mismatched ]");
                                break;
                            case ')':
                                if( $_prev_delimiter != '(' )
                                    throw new Doctrine_Excpetion("Mismatched )");
                                break;
                            default:
                                trigger_error("FATAL ERROR: UNRECOGNIZED CLOSE DELIMITER TOKEN '{$_token}' IN " . __CLASS__ . '::' . __METHOD__, E_USER_ERROR);
                        }

                        if( count($_stack) == 0 )
                            $_state = 'SECTION';
                    }
                    elseif( preg_match("#^({$open_delimiters})$#i", trim($_token)) )
                    {
                        array_push($_stack, trim($_token));
                    }

                    break;
                case 'STRING':

                    $query_parts[$_current_part] .= $_token;

                    if( preg_match("#^({$string_delimiters})$#i", trim($_token)) )
                    {
                        if( trim($_token) == end($_stack) )
                        {
                            array_pop($_stack);
                            $_state = 'SECTION';
                        }
                    }

                    break;
            }
        }
        unset($item, $_current_part, $_token, $_stack, $_state, $_section);

        //DIVIDE UP THE SELECT STATEMENT TO PREPARE TO INSERT THE ROW_NUMBER() SELECT FIELD
        $_select_split = array_map('trim', preg_split(
            "#^(DISTINCT|)[ ]*(TOP[ ]+[0-9]+|)#i",
            trim($query_parts['SELECT']),
            -1,
            PREG_SPLIT_DELIM_CAPTURE | PREG_SPLIT_NO_EMPTY
        ));

        $_select_details = array_pop($_select_split);

        $query_parts['SELECT'] = array_merge((array)implode(' ', $_select_split), (array)$_select_details);
        unset($_select_split, $_select_details);

        

        //SETUP OUTER QUERY SELECT STATEMENT
        $outer_select = array();
        foreach( array_map('trim', explode(',', end($query_parts['SELECT']))) as $_select )
        {
            $matches = array();
            if( preg_match('#AS[ ]+(?<alias>.*)$#i', $_select, $matches) )
            {
                $outer_select[] = "[{$inner_query_name}]." . trim($matches['alias']);
            }
            else if( preg_match('#^(?<table>(\[[^\]]+\]|[^\.]+)\.|)(?<field>.*)#i', $_select, $matches) )
            {
                $outer_select[] = "[{$inner_query_name}]." . trim($matches['field']);
            }
        }

        //SETUP ROW_COUNT OVER() SEGMENT
        if( isset($query_parts['ORDER BY']) )
        {
            $row_count_select = "Row_Number() OVER (ORDER BY " . $query_parts['ORDER BY'] . ") AS [{$row_count_name}]";
            unset($query_parts['ORDER BY']); //ORDER BY NOT ALLOWED IN SUBQUERY, OVER(...) TAKES ITS PLACE
        }
        else
        {
            $row_count_select = "Row_Number() OVER (ORDER BY (SELECT 0)) AS [{$row_count_name}]";
        }

        $query = implode(' ', array(
            'SELECT',
            count($query_parts['SELECT']) > 1 ?
                $query_parts['SELECT'][0] . ' ' . implode(', ', array(
                    $row_count_select,
                    $query_parts['SELECT'][1]
                )) :
                implode(', ', array(
                    $row_count_select,
                    $query_parts['SELECT'][0]
                )),
        ));

        unset($query_parts['SELECT']);
        foreach( $query_parts as $section => $parameters )
            $query .= ' ' . $section . ' ' . $parameters;


        $outer_query = "SELECT " . implode(', ', $outer_select) . " FROM (" . $query . ") AS [{$inner_query_name}]";

        if( $limit || $offset )
        {
            $outer_where = array();
            if( $limit )
                $outer_where[] = "[{$inner_query_name}].[{$row_count_name}] <= " . ($limit + $offset);
            if( $offset )
                $outer_where[] = "[{$inner_query_name}].[{$row_count_name}] > " . $offset;

            $outer_query .= ' WHERE ' . implode(' AND ', $outer_where);
        }
        
        return $outer_query;
    }

    public function is2005OrBetter()
    {
        if( !isset($this->is2005OrBetter) )
        {
            $version = $this->getServerVersion();

            if( $version['major'] >= 9 )
                $this->is2005OrBetter = true;
            else
                $this->is2005OrBetter = false;
        }

        return $this->is2005OrBetter;
    }

    /**
     * return version information about the server
     *
     * @param bool   $native  determines if the raw version string should be returned
     * @return array    version information
     */
    public function getServerVersion($native = false)
    {
        if ($this->serverInfo) {
            $serverInfo = $this->serverInfo;
        } else {
            $query      = 'SELECT @@VERSION';
            $serverInfo = $this->fetchOne($query);
        }
        // cache server_info
        $this->serverInfo = $serverInfo;
        if ( ! $native) {
            if (preg_match('/([0-9]+)\.([0-9]+)\.([0-9]+)/', $serverInfo, $tmp)) {
                $serverInfo = array(
                    'major' => $tmp[1],
                    'minor' => $tmp[2],
                    'patch' => $tmp[3],
                    'extra' => null,
                    'native' => $serverInfo,
                );
            } else {
                $serverInfo = array(
                    'major' => null,
                    'minor' => null,
                    'patch' => null,
                    'extra' => null,
                    'native' => $serverInfo,
                );
            }
        }
        return $serverInfo;
    }
}
```

