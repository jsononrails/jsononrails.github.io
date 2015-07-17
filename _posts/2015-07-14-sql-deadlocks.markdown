---
layout: post
title:  "SQL Deadlocks"
date:   2015-07-14 14:34:25
categories: [sql, deadlocks]
tags: [sql server, deadlocks, dot net, C Sharp]
image: /assets/images/windows.jpg
summary: How to deal with SQL deadlocks
comments: true
published: true
---
Recently while working on a real time web application at work we discovered that a select query in one stored procedure was executing at the same time another was doing and insert/update to the same table. This of course created a situation known 
as a deadlock and our application began to malfunction.

The solution was to update our stored procedure to include<br />
`SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED` see below.

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

This allows for uncommitted reading of all tables queried within the stored procedure. If you need more granularity then I'd suggest using `NOLOCK` 
only to the table(s) in question individually.

Then in our database repository where we call the stored procedure I wrap the call in a `try catch` to enforce a retry if the database throws a `deadlock` or `timeout` exception

<em>The following is C# code</em>
{% highlight csharp %}
public bool MyFunction(var var1, var var2)
{
    int retryCount = 3;
    bool success = false;
    
    try
    {
    	while (retryCount > 0 && !success)
    	{
    		// do database stuff
    		
    		// on success
    		success = true;
    	}
    }
    catch (SqlException e)
    {
    	if (e.Number == 1205) // SQL deadlock exception
    	{
    		// log exceptions
    		
    		// decrement retry count
    		retryCount--;
    	}
    	else if (e.Number == -2) // SQL timeout exception
    	{
    		// log exceptions
    		
    		// decrement retry count
    		retryCount--;
    	}
    
    	// log exception
    	
    	return false;
    }
    catch (Exception e)
    {
    	// log exception
    	
    	return false;
    }
    
    return true;
}
{% endhighlight %}

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
