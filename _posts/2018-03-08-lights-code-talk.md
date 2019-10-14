---
layout: post
title: Code Talk - How the Test Lights Work on Learn.co
tags: [code reading]
---

## aka The Little Submission That Could

by [@talum](https://github.com/talum)

Submissions take a long, multi-step journey from [`learn-co` gem](https://github.com/learn-co/learn-co) to Learn.co.

### Step 1: [`learn-co` gem](https://github.com/learn-co/learn-co)

Collection of open source gems (one parent gem with buncha child gems)

Focus for this talk is [`learn-test`](https://github.com/learn-co/learn-test)  
  - figures out what kind of test strategy to use
  - runs tests
  - turns test results into JSON payload
  - show student human readable results (written to `results.json`)
  - sends payload to Ironbroker V2
    - user info
    - repo info
    - test success/failure info

### Step 2: Ironbroker V2

Message broker app written in Sinatra.

Handles all webhooks from variety of sources  
  - FlatironUnitTest
  - FlatironRspec
  - GitHub
  - ScheduleOnce

By "handle", specifically this means process and publish to topic exchange  
 - parse
 - save
 - validate
 - resolve event name
 - resolve ampq topic
 - resolve ampq routing key
 - publish
 - handle bugs
 - publish trackable
 - save (again)


### Step 3: [RabbitMQ](https://www.rabbitmq.com)

Message broker over AMQP.

Payload arrives at topic exchange, then routed to specific queue (based on routing key). In this example, the `local_build` queue.


### Step 4: Worker

Messages are peeled off by consumers (powered by [Sneakers](https://github.com/jondot/sneakers), a high performance RabbitMQ background processing framework for Ruby, aka "the best RabbitMQ processor for Ruby").

We have Sneakers-backed workers that subscribe to queues then pull off and process messages.

Passed from worker to services:

`LocalBuildWorker` (AMQP consumer) => `LocalBuildSubmissionHandler` => `SubmissionHandler`


### Step 5: Service

`SubmissionHandler`  
  - core service that receives all Submission payloads for processing
  - currently doing too much. high level:
    - records submission and progress
    - publishes activity to activity feed
    - publishes a new payload to Pushstream
      - PushstreamPublisher
      - PushstreamWorker
      - PushstreamGateway

On the backend, user now has credit for this submission (aka progress records have been updated), but front-end light still hasn't flipped.


### Step 6: Frontend Scripts for Pushstream pub sub

Every user on Learn.co gets a subscription to a channel for their user on Pushstream.

Script sets listener for `submission` Pushstream events.

On `submission` event, script checks payload and if passing, flips light green.


### DETOUR: Debugging

**Question:** Did my message make it to RabbitMQ?  
**Answer:** Look at Sneakers log (`tail -f log/sneakers.log | grep -v eartbeat`)

**Question:** Did the right worker get called?  
**Answer:**
  - Can't `binding.pry` in a worker
  - Instead `puts` out message and `p` the payload (`p` => `puts` and `inspect`)
  - Now that you have the payload, you can open up Rails console and pass it to whatever service you wanna test. Run the service synchronously and `binding.pry` as much as you want w/i service.


### WHAT IF you want to run everything locally for testing/development?

1. Start up `phoeyonce` and get Docker container id
2. Log into container as test user, start bash
3. Run Ironbroker v2 locally
   - Start localtunnel for that port so you can connect to it from Docker container
4. Open `learn-test` gem, modify SERVICE_URL to localtunnel url
5. Start `ironboard` locally
6. Start workers locally

EASY PEASY.


### Resources

1. WeWork video about RabbitMQ
2. [`learn-co` gem](https://github.com/learn-co/learn-co)
