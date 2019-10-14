---
layout: post
title: Phil Sturgeon - A No-Nonsense GraphQL and REST Comparison
tags: ['code reading', 'graphql']
---

Notes from Phil Sturgeon's code talk at Flatiron School, 3/15/2018.

Slides available at [`philsturgeon.uk/speaking`](https://talks.philsturgeon.uk/instances/rubyconfco-17/a-no-nonsense-graphql-rest-comparison/html/).

### What is GraphQL?

GraphQL = FQL + (REST - [Hypermedia](https://en.wikipedia.org/wiki/Hypermedia))

A little FQLish, a little RESTish.

### What is REST?

- REST != CRUD API
- REST == state machines over HTTP
- Nice wrapper for models (keep logic in controller)
- State machines can power [Hypermedia](https://en.wikipedia.org/wiki/Hypermedia) controls (see below)

### What is [Hypermedia](https://en.wikipedia.org/wiki/Hypermedia)?

- State machines over JSON
- Misused
- Misunderstood
- Hypermedia links are not just for related data

More on [HATEOS](https://spring.io/understanding/HATEOAS):

Level 1: string containing URL  
- no guarantee of success
- make url respond to options

Level 2: add metadata to options payload  
- use JSON schema to detail fields
- use [JSON Hyperschema](http://json-schema.org/latest/json-schema-hypermedia.html) to detail potential actions
- plus another optional layer of strictness
- might to ignored/unnoticed by clients

Level 3: Add Hypermedia controls in response  
- Siren / HAL
- Just send back everything (but end up with potentially huge response size)


#### Other protocols

- SPARQL (2008) - not popular
- FIQL (2008) - flexible, but not popular


#### Some things GraphQL is bad at

- Caching
  - Query langagues are bad at caching
  - Each client has to guess rules for when and how to invalidate
  - Guessing rules for caching + how to invalidate, not ideal
  - REST says caching should be the server's concern
  - Alternatives:
    - Endpoint-based APIs can utilize HTTP caching
      - super powerful
    - Network caching:
      - Varnish - sits in front of application server, all requests go through it
      - nginx

- GraphQL is only as performant/efficient as the database you're querying
  - Data has to be readily accessible for any query
  - Forces you to optimize database architecture
  - GraphQL is only efficient/performant as the database it's querying
  - Watch out for the MEGA-INCLUDES problem (if people can ask for everything, they will)

Unlike HATEOAS, GraphQL can't help you communicate with other systems (can't do cross-API requests)


#### Some things GraphQL is good at

- Customizable APIs
- Can be more performant that REST (if used wisely)
- Middleware: [`faraday-http-cache`](https://github.com/plataformatec/faraday-http-cache) to magically respect cache headers
- Can use fields, partials
  - But be aware: these custom queries completely ruin HTTP network caching tools

- Handy when bulding JSONish / RESTish APIs

- Makes deprecations easier
  - Easy to know who is requesting what fields (can infer from request logs).
  - Deprecate entire endpoing using sunset protocol (see [`faraday-sunset`](https://github.com/wework/faraday-sunset))

### Before switching, questions to ask yourself:

- How different are your clients from one another?
- Do you never want to use HATEOS? (hard to switch back)
 - Do you need to support file uploads?

### Small Wins

- [JSON Hyperschema](http://json-schema.org/latest/json-schema-hypermedia.html) is a good place to start before committing entirely.
- nginx now supports HTTP2. Allows multiplexing requests.
- Add cache control middleware.


## More Resources

- Slides available at [`philsturgeon.uk/speaking`](https://talks.philsturgeon.uk/instances/rubyconfco-17/a-no-nonsense-graphql-rest-comparison/html/)
- More APIs: [`blog.apisyouwonthate.com`](blog.apisyouwonthate.com)
- [JSON Hyperschema](http://json-schema.org/latest/json-schema-hypermedia.html)
- Caching middleware: [`faraday-http-cache`](https://github.com/plataformatec/faraday-http-cache)
- Sunset protocol: [`faraday-sunset`](https://github.com/wework/faraday-sunset)
