---
layout: post
title: "When to Use Curlies in Angular Directives"
date: "2013-06-24"
comments: true
categories: [Code, js, angularjs]
description: Angularjs has some sweet built-in directives.  Sometimes it can be confusing when to use the curly braces around expressions in directive attributes and when not to.  Here are a couple rules.
keywords: js, javascript, angular, angular directives, angular direct curlies, curly braces
published: true
---

Angularjs has some sweet built-in directives.  Sometimes it can be confusing when to use the curly braces around expressions in directive attributes and when not to.  Here are a couple rules.

![Angular directive curlies](http://i.imgur.com/wVBKD.png)

<!--more-->

## Curlies

By "curlies", I mean curly braces.  The difference of when to use or not use them looks like the following.

Use them:
{%codeblock lang:html%}
{%raw%}
<element ng-attribute="{{valWithCurlies}}" />
{%endraw%}
{%endcodeblock%}

Or despise them:
{%codeblock lang:html%}
{%raw%}
<element ng-attribute="valWithNone" />
{%endraw%}
{%endcodeblock%}

## Two Rules of Thumb

Here are two rules that will serve you well when deciding whether or not to use curlies.

1. Assume no curlies in Angular directive attribute values.  Most directives don't use them.
2. If you would conceivably want to mix strings with Javascript code in the attribute, use curlies.

## Two Examples with Curlies

There are only a few examples of directives using curlies in their attributes.  These examples are both relating to pathing.  Apparently that's a common use case for mixing strings with JavaScript-borne values.

### ngSrc

As in the source of an image, such as:

{%codeblock lang:html%}
{%raw%}
<img ng-src="/images/{{tool.icon_name}}.png" />
{%endraw%}
{%endcodeblock%}

### ngHref

As in a link to an html page, such as:

{%codeblock lang:html%}
{%raw%}
<a ng-href="http://google.com?q={{query}}">See Google Results</a>
{%endraw%}
{%endcodeblock%}

## Something more Precise?

Is there something more precise than these two rules of thumb?  Why does Angular make an exception for these two cases, thus causing us to try to remember our rules of thumb?

As I surmised above, I can only guess that because the pathing use case often wants to mix strings with JavaScript values, the exception was made.
