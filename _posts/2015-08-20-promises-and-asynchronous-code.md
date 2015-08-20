---
layout: post
title: Promises and Asynchronous Code
---

# Notes from 8/20/2015 code reading

![Callback Hell]({{ site.baseurl }}/assets/callback-hell.jpg "Callback Hell")

Promises are a pattern for managing async callbacks.

Calling `then()` generates another promise object, when then resolves at the end of the chain. When promise object resolves, it resolves with the data coming back, which can then be passed to the next promise in the chain. Helps maintain the idea of a return value to asynchronous code - brings return values back.

jQuery has promises built in, but not fully up to true A+ promise spec.

Another more robust library: when.js

When.js library provides many additional helper methods for promises. You can call `return when.promise(function(resolve, reject){...})` to return a promise object and pass `resolve` and `reject` functions into the promise chain.

Use `tap()` to add in something asynchronously to the chain.  
Use `catch()` at the bottom of the chain to handle an error at any point in the chain.  
Use `delay()` to add in a delay at any point.  
Use `timeout()` to throw an error if promise isn't resolved after a certain amount of time.
Use `with()` to pass in a context (usually `this`).

`When.map()` accepts an array of values or promises, and you can pass in an iterator function. It'll pass each element in the array to the function and return an array of return values. It makes every request as fast as possible, so more efficient than Ruby.

Many When.js functions mirror / have the same names and/or functionality as underscore.js and Ruby::Enumerable methods. This "standard interface" is extremely easy to work with.
