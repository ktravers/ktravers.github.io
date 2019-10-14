---
layout: post
title: Code Talk - Refactoring Retriable Actions
---

A Story About Asynchronous Actions, Dynamic Workers and Queues, and RabbitMQ.

To standardize running asynchronous services, we've created a number of "Base" objects:  
  - `AsyncBaseService`, just introduced (inherits from BaseService)
  - `BaseWorker`

Worker queues are managed by RabbitMQ.

## Problem to solve:

- Sometimes services fail and need to be retried
- We need a reliable way to retry jobs, keep records of attempts, eventually fail after a specified number of attempts, and log error on max-out failure
- To fulfill above, we built our own Retriable system, which has done more harm than good

## New solution:

- New queues and workers are spun up dynamically for each `AsyncBaseService`-backed service
- Safety through encapsulation: if one service goes sideways, problems won't impact any other queues
- Three queues for each service: standard queue, retry queue, error queue
- Config with SNEAKERS_DEFAULT_OPTIONS

Replacing Retriable system with built-in RabbitMQ tooling, most importantly, Dead Letter Exchanges. With Dead Letter Exchanges, messages can be passed to another queue on failure (aka our retry queue).

When message is rejected in original queue, gets pushed to retry queue. Retry queue waits (10s TTL), then passes back to original queue. Every message will be retried 5 times, on 6th failure gets pushed to error queue.

## Challenges

Pre-loading service files in Sneakers initializer in order to dynamically generate on runtime.

Running all workers in a single Linux process (attempting to break this up into multiple processes). Don't want workers to become a bottle-neck.


## Rollout Plan

Ship changes in pieces, proceed with caution.


## Open Questions

What do we do with messages in error queue? Currently will sit there indefinitely. We need to decide on a strategy for what to do with these (surface them somewhere, re-run them from time to time, build an interface for them, etc).

## Additional Notes

Foreman generates scripts for Linux upstart.

Worker scripts in production live in `/etc/init/sneakers`


## Resources:

- [RabbitMQ Dead Letter Exchanges](https://www.rabbitmq.com/dlx.html)
- [Sneakers](https://github.com/jondot/sneakers)
