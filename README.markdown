# SymQL
Version: 0.6   
Author: [Nick Dunn](http://nick-dunn.co.uk)  
Build Date: 2010-01-05  
Requirements: Symphony 2.0.7

## Rationale
Querying entries within your own PHP applications can be a hit-and-miss affair. Building the JOINs of field tables is time consuming, but a necessary evil even if you want to use the `EntryManager` class (which requires you to pass JOINs and WHERE strings of SQL to it). SymQL continues what [DatabaseManipulator](http://github.com/yourheropaul/databasemanipulator/) started, and is intended as a full replacement.

Essentially SymQL is a wrapper for the `EntryManager` class. It shares many similarities with Data Sources and it uses the same "filter" syntax to compile its WHERE queries. It adds additional functionality beyond Data Sources in that it allows you to perform OR queries between field (e.g. `WHERE (name='Alistair' OR name='Allen') AND published='yes'`).

The primary aim of this extension is to provide human-readable object-oriented access to Symphony entries to make building custom data sources a whole lot easier.

## Usage
Include the SymQL class:

	require_once(EXTENSIONS . '/symql/lib/class.symql.php');

Build a new query:

	$query = new SymQLQuery();

The following methods can be called on the query:

### `select`
Specify the fields to return for each entry. Accepts a comma-delimited string of field handles or IDs. For example:

	$query->select('*'); // all fields

	$query->select('name, body, published'); // field handles

	$query->select('name, 4, 12, comments:items'); // mixture of handles, IDs and fields with "modes" e.g. Bi-Link

	$query->select('system:count'); // count only

### `from`
Specify the section from which entries should be queried. Accepts either a section handle or ID. For example:

	$query->from('articles');

### `where`
Build a series of select criteria. Use DS-filtering syntax. Accepts a field (handle or ID), the filter value, and an optional type.For example:

	$query->where('published', 'yes'); // filters a checkbox with a Yes value

	$query->where('title', 'regexp:CSS'); // text input REGEX filter

As with a Data Source, filters combine at the SQL level with the AND keyword. Therefore using the two filters above would select entries where published=yes AND title is like CSS. If you want the filters to concatenate using the OR keyword, pass your preference as the optional third argument:

	$query->where('title', 'regexp:Allen');
	$query->where('title', 'regexp:Alistair', SymQL::DS_FILTER_OR);
	$query->where('published', 'yes', SymQL::DS_FILTER_AND);

The above will find published entries where the title matches either Nick or Dunn. Note that the default is `SymQL::DS_FILTER_AND` so this doesn't need to be set explicitly on the third `where` above.

System parameters can also be filtered on, as with a normal Data Source:

	$query->where('system:id', 15);

	$query->where('system:date', 'today);

### `orderby`
Specify sort and direction. Defaults to `system:id` in `desc` order. Use a valid field ID, handle or system pseudo-field (system:id, system:date). Direction values are 'asc', 'desc' or 'rand'. Case insensitive. 

	$query->orderby('system:date', 'asc');

	$query->orderby('name', 'DESC');

### `perPage`
Number of entries to limit by. If you don't want to use pagination, use this as an SQL `LIMIT`.

	$query->perPage(20);

### `page`
Which page of entries to return.

	$query->page(2);

## Run the query!
To run the SymQLQuery object, pass it to the SymQL's `run` method:

	$result = SymQL::run($query); // returns an XMLElement of matching entries

SymQL can return entries in four different flavours depending on how you want them. Pass the output mode as the second argument to the `run` method. For example:

	$result = SymQL::run($query, SymQL::RETURN_ENTRY_OBJECTS); // returns an array of Entry objects

* `RETURN_XML` (default) returns an XMLElement object, almost identical to a Data Source output
* `RETURN_ARRAY` returns the same structure as RETURN_XML above, but the XMLElement is converted into a PHP array
* `RETURN_RAW_COLUMNS` returns the raw column values from the database for each field
* `RETURN_ENTRY_OBJECTS` returns an array of Entry objects, useful for further processing

When using the default `RETURN_XML` type, the root element is named `symql` by default. To change this, pass the root element name when constructing the query:

	$query = new SymQLQuery('element-name-here');

## A simple example
Try this inside a customised Data Source. The DS should return $result, which by default is returned from SymQL as an XMLElement.

	// include SymQL
	require_once(EXTENSIONS . '/symql/lib/class.symql.php');
	
	// create a new SymQLQuery
	$query = new SymQLQuery('published-articles');
	$query
		->select('title, content, date, publish')
		->from('articles')
		->where('published', 'yes')
		->orderby('system:date', 'desc')
		->perPage(10)
		->page(1);
		
	// run it! by default an XMLElement is returned
	$result = SymQL::run($query);

## Debugging
Basic debug information can be ascertained by calling `SymQL::getDebug()` after running your query. It returns an array of query counts and SQL fragments.

	var_dump(SymQL::getDebug());die;

### Known issues
* none as of 0.6

### Changelog

* 0.6, 05 January 2010
	* renamed the AND/OR enumerators for clarity
	* added ability to change the root element when XML is returned

* 0.5, 11 December 2009
	* fixed driver PHP syntax error (thanks brendo)
	* improved conversion of XMLElement to array for RETURN_ARRAY output
	* SymQL maintains a cached reference to resolved sections/fields to reduce query counts

* 0.4, 08 December 2009
	* fixed bug that prevented multiple filters on the same field
	* tidied private variable naming
	* added debug logging for query counts and SQL fragments

* 0.3, 07 December 2009
	* SymQL is now a static class

* 0.2, 07 December 2009
	* changed `count` to `select:count` to avoid naming conflicts
	* renamed RETURN_FIELDS to RETURN_ENTRY_OBJECTS and RETURN_RAW to RETURN_RAW_COLUMNS for clarity
	* updated documentation
	* fixed extension.driver.php

* 0.1, 05 December 2009
	* initial release. Woo! Yay! Fanfare.