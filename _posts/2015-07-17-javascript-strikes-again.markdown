---
layout: post
title:  "JavaScript Strikes Again"
date:   2015-07-17 16:17:25
categories: [sql, deadlocks]
tags: [web, javascript, html, featured]
image: /assets/images/windows.jpg
summary: Explicit Object Property Declarations
comments: true
published: true
---
Today I discovered a descrepency with my JavaScript code that because it was working in FireFox I assumed it was
also working in Internet Explorer and Chrome as well.

See the following example:
{% highlight js %}
var
	firstName = 'John',
	lastName = 'Smith',
	phoneNumber = '123-123-1234',
	myObject = {};
	
	myObject.push({ firstName, lastName, phoneNumber });
{% endhighlight %}
	
	In FireFox this seems to behave as expected either it doesn't care or need a property name to be defined 
	or it smartly default names the property to the variable name and sets its value at the same time.
	{% highlight js %}
	myObject.push({ firstName: firstName, lastName: lastName, phoneNumber: phoneNumber });
	{% endhighlight %}
	However in Internet Explorer or Chrome this completely fails and a null reference exception is thrown so I had to explicitly specify the 
	property names on the push in order to get this to work cross browser.
	
See the following updated example:
{% highlight js %}
var
	firstName = 'John',
	lastName = 'Smith',
	phoneNumber = '123-123-1234',
	myObject = {};
	
	myObject.push({ 
		firstName: firstName, 
		lastName: lastName, 
		phoneNumber: phoneNumber
	});
{% endhighlight %}
Normally I would have tested this cross browser and found this bug sooner but in this particular project our spec was 
specific to FireFox support only as the app is not a public facing site but more of an internal tool for the customer.


