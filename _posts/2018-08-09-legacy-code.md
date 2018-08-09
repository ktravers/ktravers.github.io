---
layout: post
title: Living with Legacy Javascript - Event Proxies, App Seams, and Chunking Rewrites
---

Sneak peek of [@talum](https://github.com/talum)'s upcoming talk at [EmpireJS 2018](https://2018.empirejs.org/schedule.html).


### Legacy Code

Learn.co is a really big platform. So many features.

But "old code" isn't necessarily "legacy code".

What is legacy code? Some definitions:

> Code that's "no longer in use" (check stackoverflow)

> Code we've gotten from someone else [Ed. note: But isn't that almost all our code? Not everyone on our team was here when core features were written.]

> Code without tests

Our definition for this talk:

- [X] Technologies no longer supported
- [X] Inherited from someone else
- [X] Untested


### The Story of a Codebase

Context helps you be more forgiving.

The story of learn.co codebase:

| Time     | Technology                   |
|----------|------------------------------|
| Mar 2014 | jQuery                       |
| Jan 2015 | Ember                        |
| May 2015 | Backbone + Marionette        |
| May 2016 | Rm Ember, add React + Alt.js |
| Aug 2017 | Redux                        |

How do we get all of these to play nice together?

1. Build a bridge
2. Look for a seam
3. Rewrite when time is right


### Building a Bridge

Backbone Radio <> Redux proxy: `reduxRadioProxy.js`


### Finding seams

Track nav used to be fully Backbone/Marionette. Now it's one Backbone/Marionette view that mounts and unmounts a React component.

Lots of code behind IDE open button. Could have rewritten, but instead wrote a reverse proxy: `radioReduxProxy`. Redux middleware that triggers Radio event.

Delaying rewrite to allow us to ship features faster.


### Rewrite

Got directive to build a stripped down lesson page. Took the opportunity to rewrite `lesson#show` in React.

Built side-by-side w/ existing system. One route gets React/Redux. One route gets Backbone/Marionette.

But watch out for bugs!

How to mitigate bugs:

1. Write more tests
2. Test all scenarios
3. Remove old code paths ASAP
