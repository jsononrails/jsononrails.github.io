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

{% highlight js%}
var myAppModule = (function() {
	var 
        app = {

            keepAliveSettings: {
                counter: 0,
                timerX: null,
				keepAliveInterval: parseInt($('#hid_ClientKeepAliveInterval').val()),
				keepAliveEndPoint: $('#hid_ClientKeepAliveEndPoint').val()
            },

            self: null,
			
			/*** function to initialize module ***/
            init: function () {
                
				// mantain reference to module scope of this
                self = this;
				
                // setup ajax
                $.ajaxSetup({ type: "POST", contentType: "application/json; charset=utf-8", dataType: "json" });
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