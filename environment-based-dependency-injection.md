
---
layout: post
title: "Environment-based Dependency Injection"
date: 2009-09-02 13:22
comments: true
categories: [Code, ioc, java, spring]
description: In developing an email notification system recently, we became interested in code acting differently depending on what environment we were in.  A potentiall
keywords: ioc, java, spring
published: true
---

In developing an email notification system recently, we became interested in code acting differently depending on what environment we were in.  A potentially good solution for this is environment-based dependency injection.  This means that different Spring beans, for instance, will be used depending on the environment, ie dev, test, etc.  This is useful for something like emailing, because we may not want real emails hitting the mail server in dev or test environments, but we do in prod.  With some Spring constructs, it's pretty easy.

<!--more-->

If you have property files that Spring knows about, you could put entries in like this:

```text defaultContext-dev-test.properties 
mailProvider=devTestEmailServiceImpl
```

```text defaultContext.properties
mailProvider=emailServiceImpl
```

Then, in your Spring bean context file:

```xml serviceBeanContext.xml
<alias alias="emailService" name="@{mailProvider}"/>
```

The `@{}` notation is for finding variables of that name in a properties file.  Finally, in your Java code, you can reference the alias that will be different depending on your environment:

```java
@Autowired
@Qualifier("emailService")
private EmailService emailService;
```
  
