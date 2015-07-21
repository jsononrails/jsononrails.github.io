---
layout: post
published: false
title: Client side keep alive in JavaScript
comments: true
summary: How to create a client side keep alive in JavaScript for your website
date: "2015-07-20"
categories: 
  - webdevelopment
tags: 
  - web
  - javascript
  - html
  - dotnet
  - csharp
image: windows.jpg
---


Sometimes you may find yourself in need of a keep alive to maintain an active session or a connection from your client back to your server. Today I came across a need for this in one of our projects below is my implementation of a client to server keep alive using Ajax.

{% highlight js%}
var myAppModule = (function() {
    var app = {
        keepAliveSettings: {
            counter: 0,
            timerX: null,
            keepAliveInterval: parseInt($('#hid_ClientKeepAliveInterval').val()),
            keepAliveEndPoint: $('#hid_ClientKeepAliveEndPoint').val()
        },

        self: null,

        /*** function to initialize module ***/
        init: function() {

            // mantain reference to module scope of this
            self = this;

            // setup ajax
            $.ajaxSetup({
                type: "POST",
                contentType: "application/json; charset=utf-8",
                dataType: "json"
            });
        }

            refreshSession: function() {

            myAppModule.App.keepAliveSettings.counter++;
            clearTimeout(myAppModule.App.keepAliveSettings.timerX);

            $.ajax({
                    url: myAppModule.App.keepAliveSettings.keepAliveEndPoint + '?c=' + myAppModule.App.keepAliveSettings.counter,
                    type: 'GET',
                    dataType: 'json'
                })
                .done(function(jqXHR, textStatus) {
                    // log success
                })
                .fail(function(jqXHR, textStatus) {
                    // failed
                });

            myAppModule.App.keepAliveSettings.timerX = setInterval(myAppModule.App.refreshSession, myAppModule.App.keepAliveSettings.keepAliveInterval);
        },

    };

    return {
        App: app
    };

}());
{% endhighlight %}

I store two values within hidden fields called `hid_ClientKeepAliveInterval` and `hid_ClientKeepAliveEndPoint` these store the interval to repeat the keep alive and the endpoint to our server controller action to hit. Typically I store these as Application Keys in the `web.config`.

{% highlight xml %}
<!-- Client side keep alive -->
<add key="ClientKeepAliveEndPoint" value="http://localhost:3000/KeepAlive/ClientKeepAlive/"/>
<add key="ClientKeepAliveInterval" value="30000"/> <!-- milleseconds, 30 seconds -->
{% endhighlight %}

Then in my main layout `.cshtml` I assigned the values of these keys to their respective HTML hidden inputs.

{% highlight js %}
<input type="hidden" id="hid_ClientKeepAliveEndPoint" value="@System.Configuration.ConfigurationManager.AppSettings["ClientKeepAliveEndPoint"]" />
<input type="hidden" id="hid_ClientKeepAliveInterval" value="@System.Configuration.ConfigurationManager.AppSettings["ClientKeepAliveInterval"]" />
{% endhighlight %}
