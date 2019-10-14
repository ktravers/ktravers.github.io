---
layout: post
title: Code Talk Notes - Javascript Testing and Analytics
tags: [code reading, javascript, testing]
---

## Javascript Test Updates

jsdom: npm module  
- load into node environment  
- recreate DOM  
- run inside of node using Jasmine  

Jade templating  
- require in templates  
- doesn't need to rely on Rails asset pipeline  

Spies:  
- PushStreamSpy  
- spyOnBackboneHistory  
- can have it substitute the function, wrap the function then delegate to function  
- Note: mocks, spies & stubs are all very similar (functionally, conceptually)  

Can write specs for Marionette views, Backbone models  
- run in node (`npm t`)  
- load in view (same as front end)  
- stub out fixtures  
- create view then test view  

** `eval(locus)` == `binding.pry` for node **

## Analytics

Tracking should be done client-side; inconsistent if tracked from backend  
- in our system, controller will delegate to client-side  
- concept of `current_visitor` bc we don't have a user yet  

To have access to Segment system, be sure to include analytics script call (helper method)

Use aliasing to transfer data from before user is logged in to when after user has logged in  
- data stays attached to same user  
- alias vister::user  

Sequence:  
1. alias  
2. identify (updated from current visitor traits)  
3. track  

Stats on Mixpanel  
- can create/view funnels  
- see event spreadsheet  

Problems with Wiselinks on lesson#show  
- layout never re-rendered  
- analytics chain of events doesn't fire  

** Solution? Use ajax to reset cookies **

When Wiselinks fakes a page load, it gives you an event to bind onto
