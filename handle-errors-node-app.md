---
layout: post
title: "Handle Errors in a Node App"
date: "2016-08-23"
comments: true
categories:
  - "Code"
tags:
  - "js"
  - "express"
  - "nodejs"
description: How to handle 3 types of errors in an Express API in Node.
keywords: js, javascript, express, nodejs, node, error handling, uncaughtException, middleware
published: true
image: http://i.imgur.com/1arT8Ho.jpg
---

As with any software, here you can expect the unexpected.  Node apps experience errors as well.  Let's say that an error crops in our Node API -- what should we do about it?

<!--more-->

Errors can be handled in a lot of different ways.  "Handling" in this article will essentially mean error response messaging.

#### Specific Messages are Best

When handling errors, we should do so as specifically and quickly as possible.  As an example, if we can respond to a request that causes an error with a more specific error message, this would be better than responding with a generic, catch-all message.  This first message:

```
{ errors: [{ title: 'Title field missing', source: '/data/attributes/title' }] }
```

is better than this:

```
{ errors: [{ title: 'Bad request' }] }
```

But _some_ error messaging is better than nothing.

So, starting with the most specific handling cases and going to the most generic.

## Handle Specific Business Errors

Let's say that we have an endpoint that creates a new book in our books database, requested at `POST /books`.  It validates the request body data shape to ensure sufficient and correct data is entered for new books.  Let's say that a user submits a new book without the required title field.  It would be great to have specific response that tells the client what exactly is wrong.  The controller code, in part, might look like this in [Express](https://expressjs.com/):

```js
router.post('/books', (req, res, next) => {
  const errors = validate(req.body.data)
  if (errors) {
    saveBook(req.body.data, (_, newBook) => {
      res.status(201).json({ data: newBook })
    })
  } else {
    res.status(400).json({ errors })
  }
})
```

That last line is the error response with specific error messages.  These messages are presumeably generated by the `validate` function.  We can provide a specific `bad request` response status code with a `400`.  We can provide the specific error response body, which would look like:

```
{ errors: [{ title: 'Title field missing', source: '/data/attributes/title' }] }
```

## Catch All Other Errors*

Now there are other errors that might occur that we either haven't, don't want, or will never be able to anticipate enough to provide specific handling for.  In these cases, we still want to handle the error, but we'll only be able to provide minimal value and insight into the nature of the error to clients in the responses.

Let's adjust our controller code to handle potentially errors that might happen in the book saving process.  What could happen?  Anything... like database issues with connections, constraints, locks or just something bad in our code.  In our callback, let's do something with that potential error:

```js
router.post('/books', (req, res, next) => {
  const errors = validate(req.body)
  if (errors) {
    saveBook(req.body, (err, newBook) => {
      if err return next(err) // new line

      res.status(201).json({ data: newBook })
    })
  } else {
    res.status(400).json({ errors })
  }
})
```

#### Required Error Contract

The `err` that comes back should be an `Error` or a subtype that at least follows the error contract and has the 3 required fields of `name` to identify the type of error, `message` for the human readable main issue of the error, and `stack` a string of the accurate location and stack trace of the error.

#### Express' `next` for Errors

The `next` function in [Express](https://expressjs.com/) will advance to the next middleware.  In the case of passing a non-null value to `next`, remaining normal middleware will be skipped and the first error middleware will be executed.  Error middleware functions have an arity of 4 parameters instead of the usual 3, where `err` is the first parameter.  Here's a basic catch-all error handler:

```js
app.use((err, req, res, next) => {
  res.status(500).json(serializeError(err))
})
```

`serializeError` is just going to take that and transform it into something to push across the network in the response.

```
function serializeError(err) {
  return { errors: [{ status: 500, title: err.message, stack: err.stack }] }
}
```

#### Don't Leak Stacktraces

Let's add a little something else here.  We probably don't want to leak the stack trace to our users in production, so let's protect it by detecting `NODE_ENV` and update it to be something like:

```
function serializeError(err) {
  const body = { status: 500, title: err.message }
  if (process.env.NODE_ENV !== 'production')
    body.stack = err.stack
  return { errors: [payload] }
}
```

Even better.  Generic errors handled.

## Crash On the Rest

Now note the asterisk on the previous section.  We aren't actually able to catch *all* errors in that generic "catch-all" error handler.  To illustrate, let's create two new routes.  The first is a route that does nothing but throw a new `Error`:

```js
router.get('/debug/error', (req, res, next) => {
  throw new Error('Test explosion')
})
```

The above route, if requested at `GET /debug/error`, would throw a new Error and it would be caught by the generic error handler of the previous section.  This is because `throw` delivers that new `Error` synchronously.  It stays in the context of the current call stack of the request through Express middleware.  And Express can catch the `Error` and call your first error-handling middleware.

#### Out-of-Context Errors

But we can easily break out of this context and bypass the catch-all handler entirely.  All we have to do is use a `setTimeout`.  Calling `setTimeout` will queue a new message for the event loop to, at some later tick of the clock, run the enclosed function.  Within that function, we'll throw another `Error`:

```js
router.get('/debug/crash', (req, res, next) => {
  setTimeout(_ => {
    throw new Error('Error outside request context')
  }, 1000)
})
```

#### Let the Process Die

And now there's really nothing that Express can do to help us.  The `Error` will instead bubble up to the Node process running our app.  That process gives us one final opportunity to know about the occurrence of such a fatal error.  Once we know about it, we can log it.  Perhaps we can get off a call to our monitoring service.  After that, we should assume our process is compromised, potentially unstable, and just crash.  Then use a tool like [forever](https://github.com/foreverjs/forever) to detect the downed process and restart it.  Such a crash handler might look like:

```js
  process.on('uncaughtException', err => {
    log.fatal({ err }, 'uncaught exception')

    process.nextTick(_ => process.exit(1))
  })
```

The call to `process.nextTick` is meant to give some leeway for just enough processing time to finish those last ditch logging/monitoring efforts.

Now we have caught all the errors, in hopefully the best and most helpful way possible.

What other things have you done in your app to make Node error handling better?

##### Great Resources

- [Joyent Production Practices](https://www.joyent.com/node-js/production/design/errors) for all-around design practices
- [Express Error Handling](https://expressjs.com/en/guide/error-handling.html) for framework-specific understanding
- [JSON-API Error Objects](http://jsonapi.org/format/#error-objects) for sensible info to include in an error response
- [MDN Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop) for basics on the message queue in the event loop
