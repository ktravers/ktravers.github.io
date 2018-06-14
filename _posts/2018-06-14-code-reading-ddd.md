---
layout: post
title: Code Reading - Learn ❤️ Domain Driven Design
---

Challenge: introducing DDD in a legacy codebase

## What is DDD?

### Domain

"Sphere of knowledge..."


### Model

Model domain in our software


### Muggles

- Informed by people who actually do the work (stakeholders)
- Domain experts are key for bridging gap between domain and software


### Goals

- Better code
- Better product
- More happy


## What is DDD in Learn?

- It's always kind of been there
- DDD encompasses a lot of general best practices, some of which we've already been following (ex. ubiquitous language)


### First attempts: LearnApps

- Not true DDD
- Doesn't follow all the core principles


### Then DDD "Lite"

- Domains directory
- Closer to setting up microservices in our monolith
- Aggregate domain module for each domain
  - Insulation around implementation
  - Provides clearer context
  - Minimize reaching across domains
  - Primitives in, primitives out
- Still encourages collaboration
  - Not just a tool for engineers
  - Use company-wide ubiquitous language (talk to product, stakeholders)


## Resources

- Book: Eric Evans, ["Domain Driven Design"](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)
