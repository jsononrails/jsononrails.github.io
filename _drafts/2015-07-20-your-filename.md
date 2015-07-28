---
layout: post
published: false
title: ""
comments: true
summary: How to create a client side keep alive in JavaScript for your website
date: "2015-07-20"
categories: 
  - webdevelopment
tags: 
  - web
  - javascript
  - html
image: windows.jpg
---

{% highlight csharp%}
using System.Threading;
using System.Net;

 public class MvcApplication : System.Web.HttpApplication
    {
        static Thread keepAliveThread = new Thread(KeepAlive);


  
  protected void Application_Start()
        {
            versionNumber = NSVersionInfo.VersionInfo.VersionString;

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
                // About server to server keep alive
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
                        EventLogger.GetInstance().LogEvent(SEVERITY.Error, "MvcApplication:KeepAlive() server to server keep alive failed, HttpResponse code: " + response.StatusCode);
                    else
                        EventLogger.GetInstance().LogEvent(SEVERITY.Information, "MvcApplication:KeepAlive() server to server keep alive succeeded, HttpResponse code: " + response.StatusCode);

                    System.Threading.Thread.Sleep(int.Parse(ConfigurationManager.AppSettings["ServerKeepAliveInterval"]));
                }
                catch (WebException ex)
                {
                    EventLogger.GetInstance().LogEvent(SEVERITY.Error, "MvcApplication:KeepAlive() server to server keep alive failed: " + ex.Message + " " + ex.InnerException);
                    break;
                }
                catch (ThreadAbortException ex)
                {
                    EventLogger.GetInstance().LogEvent(SEVERITY.Error, "MvcApplication:KeepAlive() server to server keep alive failed: " + ex.Message + " " + ex.InnerException);
                    break;
                }
            }
        }
        
        
        
        // do this in Global.asax.cs
{% endhighlight %}