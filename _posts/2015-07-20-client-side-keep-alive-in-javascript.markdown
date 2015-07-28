---
layout: post
published: true
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
  - dot net
  - c sharp
  - featured
image: /assets/images/windows.jpg
postnumber: 3
---



Sometimes you may find yourself in need of a keep alive to maintain an active session or a connection from your client back to your server. Today I came across a need for this in one of our projects below is my implementation of a client to server keep alive using Ajax.

{% highlight js linenos %}
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
            
            // start keep keepAliveSettings
            self.refreshSession();
            
        },

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
      }

    };

    return {
        App: app
    };

}());
{% endhighlight %}

I store two values within hidden fields called `hid_ClientKeepAliveInterval` and `hid_ClientKeepAliveEndPoint` these store the interval to repeat the keep alive and the endpoint to our server controller action to hit. Typically I store these as Application Keys in the `web.config`.

{% highlight xml  linenos %}
<!-- Client side keep alive -->
<add key="ClientKeepAliveEndPoint" value="http://localhost:3000/KeepAlive/ClientKeepAlive/"/>
<add key="ClientKeepAliveInterval" value="30000"/> <!-- milleseconds, 30 seconds -->
{% endhighlight %}

Then in my main layout `.cshtml` I assigned the values of these keys to their respective HTML hidden inputs.

{% highlight js  linenos %}
<input type="hidden" id="hid_ClientKeepAliveEndPoint" value="@System.Configuration.ConfigurationManager.AppSettings["ClientKeepAliveEndPoint"]" />
<input type="hidden" id="hid_ClientKeepAliveInterval" value="@System.Configuration.ConfigurationManager.AppSettings["ClientKeepAliveInterval"]" />
{% endhighlight %}

Then include the module and kick off the keep alive.
{% highlight js  linenos %}
<script>
    $(function() {
      
        var myApp = myAppModule;
        
        myApp.init();
      
    });
</script>
{% endhighlight %}

How it works is once the module is initialized it calls the first iteration of the `refreshSession()` function.

This function increments a counter just for loggin purposes and clears any existing instance of the setInterval
timer so we don't end up with multiple instances on each new iteration.  

We then setup our Ajax call with our
values from our `web.config` logging success appropriately.

Finally we setInterval and tell it to call the
`refreshSession()` function again and assign it to timerX so we can clear it on the next pass.

NOTE: this example assumes you have jQuery referenced in you markup in order for this to work and call document ready 
`$(function() {});`