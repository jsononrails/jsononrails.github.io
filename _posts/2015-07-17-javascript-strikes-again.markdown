---
layout: post
title: JavaScript Strikes Again
date: 2015-07-17 16:34:25
categories: 
  - web
  - javascript
tags: 
  - web
  - javascript
  - html
  - firefox
  - chrome
  - internet explorer
  - featured
image: /assets/images/rails.jpg
summary: Explicit Object Property Declarations
comments: true
published: true
---


Today I discovered a discrepancy  with my JavaScript code and because it was working in Firefox I assumed it was
also working in Internet Explorer/Chrome as well.

See the following example:
{% highlight js %}
var
	firstName = 'John',
	lastName = 'Smith',
	phoneNumber = '123-123-1234',
	myArray[] = {};
	
	myArray.push({ firstName, lastName, phoneNumber });
{% endhighlight %}
	
In Firefox this seems to behave as expected. Either it doesn't care or need a key name to be defined or it smartly default names the key to the variable name and sets its value at the same time.

Example:
{% highlight js %}
myArray.push({ firstName: firstName, lastName: lastName, phoneNumber: phoneNumber });
{% endhighlight %}
	
However in Internet Explorer/Chrome this completely fails and a null reference exception is thrown so I had to explicitly specify the key names for their respective vaule pairs on the push in order to get this to work cross browser.
	
See the following updated example:
{% highlight js %}
var
	firstName = 'John',
	lastName = 'Smith',
	phoneNumber = '123-123-1234',
	myArray = {};
	
	myArray.push({ 
		firstName: firstName, 
		lastName: lastName, 
		phoneNumber: phoneNumber
	});
{% endhighlight %}

Normally I would have tested this cross browser and found this bug sooner. However for this particular project our spec was 
specific to Firefox support only, as the app is not a public facing site but more of an internal tool for our customer.
