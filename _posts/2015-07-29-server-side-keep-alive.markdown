---
layout: post
published: true
title: Server Side Keep Alive
comments: true
summary: How to create a server side keep alive in .Net MVC
date: "2015-07-29"
categories: 
  - webdevelopment
tags: 
  - web
  - dot net
  - c sharp
  - featured
image: /assets/images/windows.jpg
postnumber: 4
---

In Post #3 I showed how to create a client to server keep alive using JavaScript and jQuery.
In this post I will show how I implemented a server side keep alive to maintain an active session for
a web application where the client may be inactive for a extended period of time.

This example is written using C# and .Net MVC add the following to your `Global.asax`

{% highlight csharp%}

using System.Threading;
using System.Net;

public class MvcApplication : System.Web.HttpApplication
{
    static Thread keepAliveThread = new Thread(KeepAlive);

    protected void Application_Start()
    {
        if (bool.Parse(ConfigurationManager.AppSettings["EnableServerToServerKeepAlive"]))
        {
            // start server to server keep alive
            keepAliveThread.Start();
        }
    }
    
    protected void Application_End()
    {
        if (bool.Parse(ConfigurationManager.AppSettings["EnableServerToServerKeepAlive"]))
        {
            // abort server to server keep alive
            keepAliveThread.Abort();
        }
    }
   
    /// <summary>
    /// Server to Server keep alive
    /// </summary>
    static void KeepAlive()
    {
        while (true)
        {
            try
            {
                string url = ConfigurationManager.AppSettings["ServerKeepAliveEndPoint"] + "?c=" + Guid.NewGuid().ToString().Replace("-", "");

                HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();

                if (response.StatusCode != HttpStatusCode.OK)
                    // log failed keep alive
                else
                    // log successful keep alive
                    
                    System.Threading.Thread.Sleep(int.Parse(ConfigurationManager.AppSettings["ServerKeepAliveInterval"]));
            }
            catch (WebException ex)
            {
                // log exception
                break;
            }
            catch (ThreadAbortException ex)
            {
                // log exception
                break;
            }
        }
    }
}
{% endhighlight %}

First I import the following namespaces `System.Threading` and `System.Net` to support threading and HttpRequests.
Then I create a new thread and pass in my delegate method to be used `static Thread keepAliveThread = new Thread(KeepAlive);`

In `Application_Start` I check an application variable `EnableServerToServerKeepAlive` to see if the application should
be using the keep alive. If it is `true` I kick off the thread by calling `keepAliveThread.Start();` this will call into 
my delegate method `KeepAlive` mentioned earlier.

First thing I do here is setup a an indefinite loop this will be used to repeat our process of sending a request to the server.
I then get the url for the keep alive from the `web.config` in this case it is to a controller and action `/KeepALive/Ping` within the app itself that 
just returns a value of `true`. I also create two objects for handling the HttpRequest to the server and 
the HttpResponse.

`HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);` and `HttpWebResponse response = (HttpWebResponse)request.GetResponse();`
the `(HttpWebRequest)WebRequest.Create(url);` method takes a parameter for the url I retrieved earlier from the `web.config` and creates a HttpRequest to that endpoint.
`(HttpWebResponse)request.GetResponse();` will get the response back from the request where I can then check its success by investigating
its status code `if (response.StatusCode != HttpStatusCode.OK)`.

If the request is successful I sleep the thread based on a value stored in a variable `ServerKeepAliveInterval` in the web.config
this will delay the next iteration where the process then starts over again.

Something to note, on my first initial pass I attempted the above using `System.Net.NetworkInformation.Ping` but because my application
is using a port other than 80 my attempt failed. `Ping` doesn't support port numbers and for security reasons leaving our apps on a default port of 80 is not an option
because of this I had to revamp my solution as written above.