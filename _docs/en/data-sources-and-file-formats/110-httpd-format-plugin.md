---
title: "HTTPD Format Plugin"
slug: "HTTPD Format Plugin"
parent: "Data Sources and File Formats"
---

**Introduced in release:** 1.9

This plugin is based on the [logparser](https://github.com/nielsbasjes/logparser) web log parsing library and enables Drill to read and query httpd (Apache Web Server) and nginx access logs natively.

## Configuration
There are five fields which you can to configure in order for Drill to read web server logs.  In general the defaults should be fine, however the fields are:

|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Option           | Description                                                                                                                                                                                                                                                             |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| logFormat        | The log format string is the format string found in your web server configuration. If you have multiple logFormats then you can add all of them in this single parameter separated by a newline (`\n`). The parser will automatically select the first matching format. |
| timestampFormat  | The format of time stamps in your log files. This setting is optional and is almost never needed.                                                                                                                                                                       |
| extensions       | The file extension of your web server logs.  Defaults to `httpd`.                                                                                                                                                                                                       |
| maxErrors        | Sets the plugin error tolerance. When set to any value less than `0`, Drill will ignore all errors. If unspecified then maxErrors is 0 which will cause the query to fail on the first error.                                                                           |
| flattenWildcards | There are a few variables which Drill extracts into maps.  Defaults to `false`.                                                                                                                                                                                         |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|


```json
"httpd" : {
  "type" : "httpd",
  "logFormat" : "%h %l %u %t \"%r\" %s %b \"%{Referer}i\" \"%{User-agent}i\"",
  "timestampFormat" : "dd/MMM/yyyy:HH:mm:ss ZZ",
  "maxErrors": 0, 
  "flattenWildcards": false
}
```

## Data Model
The fields which Drill will return from HTTPD access logs should be fairly self explanatory and should all be mapped to correct data types.  For instance, `TIMESTAMP` fields are all Drill `TIMESTAMPS` and so forth. 
 
### Nested Columns
The HTTPD parser can produce a few columns of nested data. For instance, the various `query_string` columns are parsed into Drill maps so that if you want to look for a specific
 field, you can do so. 
 
 Drill allows you to directly access maps in with the format of:
 ```
<table>.<map>.<field>
```
 One note is that in order to access a map, you must assign an alias to your table as shown below:
 ```sql
SELECT mylogs.`request_firstline_uri_query_$`.`username` AS username
FROM dfs.test.`logfile.httpd` AS mylogs

```
In this example, we assign an alias of `mylogs` to the table, the column name is `request_firstline_uri_query_$` and then the individual field within that mapping is `username
`.  This particular example enables you to analyze items in query strings.  

### Flattening Maps
In the event that you have a map field that you would like broken into columns rather than getting the nested fields, you can set the `flattenWildcards` option to `true` and 
Drill will create columns for these fields.  For example if you have a URI Query option called `username`.  If you selected the `flattedWildcards` option, Drill will create a 
field called `request_firstline_uri_query_username`.  

** Note that underscores in the field name are replaced with double underscores ** 
 
 ## Useful Functions
 If you are using Drill to analyze web access logs, there are a few other useful functions which you should know about:
 
 * `parse_url(<url>)`: This function accepts a URL as an argument and returns a map of the URL's protocol, authority, host, and path.
 * `parse_query(<query_string>)`: This function accepts a query string and returns a key/value pairing of the variables submitted in the request.
 * `parse_user_agent(<user agent>)`, `parse_user_agent( <useragent field>, <desired field> )`: The function parse_user_agent() takes a user agent string as an argument and
  returns a map of the available fields. Note that not every field will be present in every user agent string. 
  [Complete Docs Here](https://github.com/apache/drill/tree/master/contrib/udfs#user-agent-functions)
 

## Implicit Columns
Data queried by this plugin will return two implicit columns:

* **`_raw`**: This returns the raw, unparsed log line
* **`_matched`**:  Returns `true` or `false` depending on whether the line matched the config string.

Thus, if you wanted to see which lines in your log file were not matching the config, you could use the following query:

```sql
SELECT _raw
FROM <data>
WHERE _matched = false
```

## Additional Functionality
In addition to reading raw log files, the following functions are also useful when analyzing log files:  

* `parse_url(<url>)`:  This function accepts a URL as an argument and returns a map of the URL's protocol, authority, host, and path.
* `parse_query( <query_string> )`:  This function accepts a query string and returns a key/value pairing of the variables submitted in the request.

A function that parses User Agent strings and returns a map of all the pertinent information is available at: https://github.com/cgivre/drill-useragent-function

