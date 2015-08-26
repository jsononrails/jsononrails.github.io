---
layout: post
published: true
title: Search Tables or Procs
comments: true
summary: How how to seach a table or stored procedure in SQL
date: "2015-08-26"
categories: 
  - sql
tags: 
  - sql
  - featured
image: /assets/images/windows.jpg
postnumber: 5
---

Often I find myself working on large databases accross multiple projects and I start forgetting where certain things are.
After having this occur multiple times I decided to look into searching the database for a keyword to see which tables or stored procedures
that item may exist in.  

There are many resources for this on using T-SQL, my two favorite links are:
[SQL SERVER – 2005 – Search Stored Procedure Code – Search Stored Procedure Text](http://blog.sqlauthority.com/2007/09/03/sql-server-2005-search-stored-procedure-code-search-stored-procedure-text/) and
[SQL SERVER – Query to Find Column From All Tables of Database](http://blog.sqlauthority.com/2008/08/06/sql-server-query-to-find-column-from-all-tables-of-database/)

After copying and pasting these over and over whenever I needed I finally decided to merge the two into a stored procedure for easier reuse.

{% highlight sql linenos %}
CREATE PROCEDURE [dbo].[spS_UTIL_SEARCH_TABLES_AND_PROCS]
	@phrase VARCHAR(150)
AS
BEGIN

/*****************************************************************************************************************
 * File: spS_UTIL_SEARCH_TABLES_AND_PROCS
 * Author: Jason McBride
 * Version 1.0
 *
 * This proc takes a parameter and searches all tables and
 * stored procedures and returns the results
 *
 * HISTORY
 *
 * Initial version 1.0
 * August 14, 2015
 *
 *****************************************************************************************************************/
 
-- search tables
SELECT t.name AS table_name,
SCHEMA_NAME(schema_id) AS schema_name,
c.name AS column_name
FROM sys.tables AS t
INNER JOIN sys.columns c ON t.OBJECT_ID = c.OBJECT_ID
WHERE c.name LIKE '%' + @phrase +'%'
ORDER BY schema_name, table_name;

-- search procs
SELECT Name
FROM sys.procedures
WHERE OBJECT_DEFINITION(OBJECT_ID) LIKE '%' + @phrase + '%'
RETURN;
END
{% endhighlight %}

The only caveat is that you need to add this proc to any database in which you wish to run the search, but then you can execute
the search anytime you want by running the following SQL.SQL

{% highlight sql linenos %}
USE YOUR_DATABASE_NAME;

GO
EXEC [spS_UTIL_SEARCH_TABLES_AND_PROCS] 'someword'
GO
{% endhighlight %}