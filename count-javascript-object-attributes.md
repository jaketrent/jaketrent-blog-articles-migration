---
layout: post
title: "Count Attributes in JavaScript Object"
date: 2012-11-29 13:11
comments: true
categories: [Code, js]
description: How do you find the number of attributes in a JS object?  Turns out there's not a fantastic way to do it
keywords: js, javascript attributes, count js attributes
published: true
---

Sometimes it may be useful to know how many attributes a JavaScript object has.  There's not a fantastic way to do this.  Or maybe there is.  Do you know one?  Could cut out IE8 support.  Or, for what it's worth, here's a way.

![Count attributes in js object](http://i.imgur.com/54smb.jpg)

<!--more-->

## Which Objects have a Length

If instantiate a new object, it does *not* have a built-in length attribute:

{% codeblock lang:js %}
> var obj = {}
undefined
> obj.length
undefined
{% endcodeblock %}

If I add an arbitrary property that object, it still doesn't affect the length:

{% codeblock lang:js %}
> obj.prop1 = "adsf"
'adsf'
> obj.length
undefined
{% endcodeblock %}

If I instantiate a new array, it has a built-in length attribute:

{% codeblock lang:js %}
> var arr = []
undefined
> arr.push(1)
1
> arr.push(2)
2
> arr.length
2
{% endcodeblock %}

But an array is an object whose attributes are numbers, right?  So, if I put an arbitrary property on it, does it affect length:

{% codeblock lang:js %}
> arr.prop1 = "asdf"
'asdf'
> arr.length
2
{% endcodeblock %}

No, this `arr` array still just has the two items we previously pushed onto it.

## Just Iterating

So, how do you find out how many attributes are in an object (not array)?  Just iterate on it:

{% codeblock lang:js %}
function numAttrs(obj) {
  var count = 0;
  for(var key in obj) {
    if (obj.hasOwnProperty(key)) {
      ++count;
    }
  }
  return count;
}
{% endcodeblock %}

I found this a bit painful to write.  There should be a better way, right?  Do you know what it is?

## A Better Way for Better Browsers

If your in a [Modern JavaScript environment](http://kangax.github.com/es5-compat-table/), give this a whirl:

{% codeblock lang:js %}
  Object.keys(obj).length
{% endcodeblock %}

Way nicer.  Don't try in <= IE8.