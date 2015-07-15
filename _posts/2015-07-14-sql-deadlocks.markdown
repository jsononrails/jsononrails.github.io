---
layout: post
title:  "SQL Deadlocks"
date:   2015-07-14 14:34:25
categories: [sql, deadlocks]
tags: [sql server, deadlocks, featured]
image: /assets/images/windows.jpg
summary: How to deal with SQL deadlocks
published: true
---
Recently while working on a real time web application at work we discovered that a select query in one stored procedure was executing at the same time another was doing and insert/update to the same table. This of course created a situation known 
as a deadlock and our application began to malfunction.

The solution was to update our stored procedure to include `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED` see below.

{% highlight sql %}
USE [YOUR_DATABASE_NAME]

GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER PROCEDURE [dbo].[spS_YOUR_PROC]
	@Var1 INT
AS
BEGIN

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

	-- do stuff

RETURN;
END
{% endhighlight %}

This allows for uncommitted reading of all tables queried within the stored procedure. If you need more granularity then I'd suggest using NOLOCK 
only to the table(s) in question individually.


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
