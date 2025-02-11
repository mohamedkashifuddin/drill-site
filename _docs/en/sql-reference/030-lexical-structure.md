---
title: "Lexical Structure"
slug: "Lexical Structure"
parent: "SQL Reference"
---

A SQL statement used in Drill can include one or more of the following parts:

* Clause, such as FROM
* Command, such as SELECT 
* Expression, a combination of one or more values, operators, and SQL functions that evaluates to a value. For example, users.firstname is a period expression.
* Function, scalar and aggregate, such as sum
* Literal value

  * [Boolean]({{ site.baseurl }}/docs/lexical-structure/#boolean)
  * [Identifier]({{ site.baseurl }}/docs/lexical-structure/#identifier)
  * [Integer]({{ site.baseurl }}/docs/lexical-structure/#integer)
  * [Numeric constant]({{ site.baseurl }}/docs/lexical-structure/#numeric-constant)
  * [String]({{ site.baseurl }}/docs/lexical-structure/#string)

* Operator, such as [NOT] IN, LIKE, and AND
* Predicate, such as a > b in `SELECT * FROM myfile WHERE a > b`.
* [Storage plugin and workspace reference]({{ site.baseurl }}/docs/lexical-structure/#storage-plugin-and-workspace-references)
* Whitespace
* Comment in the following format: 

        /* This is a comment. */


## Case-Sensitivity

Drill is case-insensitive; however, data sources may be case-sensitive. The data source determines the case-sensitivity of table and column names.  

### Data Sources  

When writing queries that reference tables or columns in case-sensitive data sources, you must reference the table or column exactly as it is stored in the data source. For example, if you query a table named “customers” in HBase, you cannot write the table name as “CUSTOMERS” in a query against the table because the data source distinguishes between upper and lower case.  

Table values referenced in a query are case-sensitive, for example:  

       SELECT * FROM t WHERE col1='a';  

If values in col1 of table t are stored in uppercase, this query would not return any results because the query requested lowercase 'a'.
  

The following lists provide the case sensitivity of some data sources that Drill supports.  

**Case-Insensitive**  

- JSON  
- Hive  
- Parquet    

**Case-Sensitive**  

- MapR Database (MapR-DB)
- HBase  
- Table values, irrelevant of data source  

### Drill  

Case-insensitivity in Drill applies to:  

- SQL functions  
- SQL command names  
- Storage plugin names (as of Drill 1.15)  
- Workspace (schema) names (as of Drill 1.15)  
- Table names in the Drill system storage plugins, which includes `information_schema` tables and `sys` tables (as of Drill 1.15)  
- Keywords, for example SELECT and select are equivalent, as shown in the following queries:  

             SELECT * FROM dfs.`/Users/drilluser/ticket_sales.json`;
             select * from DFS.`/Users/drilluser/ticket_sales.json`;  


### Storage Plugin and Workspace References

The case of the name used in a query and the name in the storage plugin definition do not have to match. For example, defining a storage plugin named `dfs` and then referring to the plugin as `DFS` or `dfs` in a query are both acceptable to Drill. For example, Drill can process both of the following queries:  

       SELECT * FROM dfs.`/Users/drilluser/ticket_sales.json`;
       SELECT * FROM DFS.`/Users/drilluser/ticket_sales.json`;  

**Note:** Drill stores storage plugin and workspace names in lowercase, by default. If you upgrade to Drill 1.15, you can query storage plugin or schema names previously stored in uppercase using uppercase or lowercase text. If duplicate names exist, for example you have two storage plugins named DFS, one in uppercase and one in lowercase, Drill logs a warning and keeps only one storage plugin. Drill keeps the first storage plugin defined in the workspace configuration.   

## Literal Values

This section describes how to construct literals.

### Boolean
Boolean values are true or false and are case-insensitive. Do not enclose the values in quotation marks.

### Date and Time
Format dates using dashes (-) to separate year, month, and day. Format time using colons (:) to separate hours, minutes and seconds. Format timestamps using a date and a time. These literals are shown in the following examples:

* Date: 2008-12-15

* Time: 22:55:55.123...

* Timestamp: 2008-12-15 22:55:55.12345

If you have dates and times in other formats, use a [data type conversion function]({{site.baseurl}}/docs/data-type-conversion/) in your queries.

### Identifiers
An identifier is a letter followed by any sequence of letters, digits, or the underscore. For example, names of tables, columns, and aliases are identifiers. Maximum length is 1024 characters. Enclose the following identifiers with identifier quotes:

* Keywords
* Identifiers that SQL cannot parse  

**Note:**  The term “user” is a reserved keyword, however if you reference a field/column named “user” in a query and you enclose the term in back ticks (\`user\`), Drill does not treat \`user\` as an identifier. Instead, Drill treats \`user\` as a special function that calls the current user. To work around this issue, use a table alias when referencing the field/column. The table alias informs the parser that this identifier is not a function call, but a regular identifier. For example, assume a table alias “t.” Use t.\`user\` instead of \`user\` as shown:

       SELECT operation, t.`user`, uid FROM `dfs`.`/drill/student` t;
 

For example, enclose the SQL keywords date and time in identifier quotes when referring to column names, but not when referring to data types:

    CREATE TABLE dfs.tmp.sampleparquet AS 
    (SELECT trans_id, 
    cast(`date` AS date) transdate, 
    cast(`time` AS time) transtime, 
    cast(amount AS double) amountm,
    user_info, marketing_info, trans_info 
    FROM dfs.`/Users/drilluser/sample.json`);    

Table and column names are case-insensitive. Use identifier quotes to enclose names that contain special characters. Special characters are those other than the 52 Latin alphabet characters. For example, space and @ are special characters. 

The following example shows the keyword Year enclosed in identifier quotes. Because the column alias contains the special space character, also enclose the alias in backticks, as shown in the following example:

    SELECT extract(year from transdate) AS `Year`, t.user_info.cust_id AS `Customer Number` FROM dfs.tmp.`sampleparquet` t;
    |------------|-----------------|
    |    Year    | Customer Number |
    |------------|-----------------|
    | 2013       | 28              |
    | 2013       | 86623           |
    | 2013       | 11              |
    | 2013       | 666             |
    | 2013       | 999             |
    |------------|-----------------|
    5 rows selected (0.051 seconds)  


### Identifier Quotes
Prior to Drill 1.11, the SQL parser in Drill only supported backticks as identifier quotes. As of Drill 1.11, the SQL parser can also use double quotes and square brackets. The default setting for identifier quotes is backticks. You can configure the type of identifier quotes used with the  `planner.parser.quoting_identifiers` configuration option, at the system or session level, as shown:  

       ALTER SYSTEM|SESSION SET planner.parser.quoting_identifiers = '"';  
       ALTER SYSTEM|SESSION SET planner.parser.quoting_identifiers = '[';  
       ALTER SYSTEM|SESSION SET planner.parser.quoting_identifiers = '`';  

The following table lists the supported identifier quotes with their corresponding Unicode character:   
 
| Quoting   Identifier | Unicode   Character                                                   |
|----------------------|-----------------------------------------------------------------------|
| Backticks            | 'GRAVE   ACCENT' (U+0060)                                             |
| Double quotes        | 'QUOTATION   MARK' (U+0022)                                           |
| Square brackets      | 'LEFT   SQUARE BRACKET' (U+005B) and 'RIGHT SQUARE BRACKET' (U+005D)  |  

Alternatively, you can set the type of identifier using the `quoting_identifiers` property in the jdbc connection URL, as shown:  
 
       jdbc:drill:zk=local;quoting_identifiers=[  

**Note:** The identifier quotes used in queries must match the `planner.parser.quoting_identifiers` setting. If you use another type of identifier quotes, Drill returns an error.  

The following queries show the use of each type of identifier quotes:  

       0: jdbc:drill:zk=local> select `employee_id`, `full_name` from cp.`employee.json` limit 1;

       |-------------|--------------|
       | employee_id | full_name    |
       |-------------|--------------|
       | 1           | Sheri Nowmer |
       |-------------|--------------|
       1 row selected (0.148 seconds)  

       0: jdbc:drill:zk=local> select "employee_id", "full_name" from cp."employee.json" limit 1;
       |-------------|--------------|
       | employee_id | full_name    |
       |-------------|--------------|
       | 1           | Sheri Nowmer |
       |-------------|--------------|
       1 row selected (0.129 seconds)  

       0: jdbc:drill:zk=local> select [employee_id], [full_name] from cp.[employee.json] limit 1;
       |-------------|--------------|
       | employee_id | full_name    |
       |-------------|--------------|
       | 1           | Sheri Nowmer |
       |-------------|--------------|
       1 row selected (0.14 seconds)  

### Dots in Column Names  
As of Drill 1.12, Drill supports dots in column names if the data source itself allows dots in column names, such as JSON and Parquet , as shown in the following example:  

       SELECT * FROM `test.json`;
       
       |-------------------------------------------------|-------------------------------------------------|
       | 0.0.1                                           | 0.1.2                                           |
       |-------------------------------------------------|-------------------------------------------------|
       | {"version":"0.0.1","date_created":"2014-03-15"} | {"version":"0.1.2","date_created":"2014-05-21"} |
       |-------------------------------------------------|-------------------------------------------------|

When referencing column names with dots in queries, you must escape the dots with [identifier quotes]({{site.baseurl}}/docs/lexical-structure/#identifier-quotes), as shown in the following query:  

       SELECT t.`0.1.2`.version FROM dfs.tmp.`test.json` t WHERE t.`0.1.2`.date_created='2014-05-21'  

Drill also supports associative array indexing, for example `SELECT a, b.c, b['d.e']...`.  

Note that in this example, the `“d.e”` field is selected, not a map `“d”` with field `“e”` inside of it.   


### Integer
An integer value consists of an optional minus sign, -, followed by one or more digits.

### Numeric constant

Numeric constants include integers, floats, and values in E notation.

* Integers: 0-9 and a minus sign prefix
* Floats: a series of one or more decimal digits, followed by a period, ., and one or more digits in decimal places. There is no optional + sign. Leading or trailing zeros are required before and after decimal points. For example, 0.52 and 52.0. 
* E notation: Approximate-value numerical literals in scientific notation consist of a mantissa and exponent. Either or both parts can be signed. For example: 1.2E3, 1.2E-3, -1.2E3, -1.2E-3. Values consist of an optional negative sign (using -), a floating point number, letters e or E, a positive or negative sign (+ or -), and an integer exponent. For example, the following JSON file has data in E notation in two records.

        {"trans_id":0,
         "date":"2013-07-26",
         "time":"04:56:59",
         "amount":-2.6034345E+38,
         "trans_info":{"prod_id":[16],
         "purch_flag":"false"
        }}

        {"trans_id":1,
         "date":"2013-05-16",
         "time":"07:31:54",
         "amount":1.8887898E+38,
         "trans_info":{"prod_id":[],
         "purch_flag":"false"
        }}
  Aggregating the data in Drill produces scientific notation in the output:

        SELECT sum(amount) FROM dfs.`/Users/drilluser/sample2.json`;

        |--------------|
        | EXPR$0       |
        |--------------|
        | -7.146447E37 |
        |--------------|
        1 row selected (0.044 seconds)

Drill represents invalid values, such as the square root of a negative number, as NaN.

### String
Strings are characters enclosed in single quotation marks. To use a single quotation mark itself (apostrophe) in a string, escape it using a single quotation mark. For example, the value Martha's Vineyard in the SOURCE column in the `vitalstat.json` file contains an apostrophe:

    |-------------------|
    | SOURCE            |
    |-------------------|
    | Martha's Vineyard |
    | Monroe County     |
    |-------------------|
    2 rows selected (0.053 seconds)

To refer to the string Martha's Vineyard in a query, use single quotation marks to enclose the string and escape the apostophe using a single quotation mark:

    SELECT * FROM dfs.`/Users/drilluser/vitalstat.json` t 
    WHERE t.source = 'Martha''s Vineyard';





