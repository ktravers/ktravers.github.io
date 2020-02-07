---
layout: post
title: Notes on Designing Data-Intensive Applications
tags: [book report]
---

I'm reading ["Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems" by Martin Kleppmann](https://learning.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) as part of a technical book club at work. We'll be covering a new chapter each week for the next ~12 weeks. Here's my notes on each chapter.

## Intro: Foundations of Data Systems

- "Data-intensive" vs "compute-intensive"
  - Data intensive if "data is [application's] primary challenge" in terms of quantity, complexity, or turnover
  - Compute intensive if CPU cycles "are the bottleneck"
  - This book focuses on data intensive

## Part I. Foundations of Data Systems

### Ch 1: Reliable, Scalable, and Maintainable Applications

- Application requrements:
  - Functional requirements: what it should do, such as allowing data to be stored, retrieved, searched, and processed
  - Nonfunctional requirements: general properties like security, reliability, compliance, scalability, compatibility, maintainability
- Goal: "reliable, scalable, and maintainable data systems"
  - Reliability:
    - System works correctly even in the face of adversity
    - Application functions as expected
    - Tolerance for mistakes/surprises (resiliancy)
      - "Fault is not the same as a failure"
      - Fault: system component deviates from its spec
        - Hardware faults are common, so you need redundancy
        - Software errors: systematic error w/i the system
        - Human errors: leading cause of outages
      - Failure: system as a whole stops providing required service to the user
      - Designer's goal: **prevent faults from turning into failures**
    - Performs well even under load (performance)
    - Prevents abuse, unauthorized access (security)
  - Scalability: systems's ability to cope with increased load (in terms of data, traffic, or complexity)
    - Scaling gets tricky when data system is stateful
    - Design for which operations will be common vs rare (load parameters)
  - Maintainability: system doesn't prevent the team who's building it from being productive
    - Operability: make life easy for your ops team
    - Simplicity: make it easy for new folks to understand the system
    - Evolvability: make it easy to make changes (extensibility, modifiability, plasticity)
- Building blocks of data intensive applications
  - Databases: store
  - Caches: remember expensive operations for faster reads
  - Search indexes
  - Stream processing (sending messages between processes for async handling)
  - Batch processing
- Hard to choose tools for the above
- Hard to combine tools
- "Composite data system" could describe pretty much every codebase I've ever worked on.
- Questions for data system designer:
  - "How do you ensure that the data remains correct and complete, even when things go wrong internally?"
  - "How do you provide consistently good performance to clients, even when parts of your system are degraded?"
  - "How do you scale to handle an increase in load?"
  - "What does a good API for the service look like?"
- Ways to prevent faults:
  - Carefully thinking about assumptions and interactions in the system
    - Minimize opportunities for human error
    - Provide non-production environments where humans can explore, experiment
  - Thorough testing: unit, integration, and manual tests
  - Process isolation
  - Allowing processes to crash and restart
  - Measuring, monitoring, and analyzing system behavior in production
  - Enable quick recovery (also mentioned in Google SRE book)

#### Questions

- What is a RAID configuration? (from section about hardware faults)
- Definitions of the ops team's responsibilities seemed very broad.
  - Preserve organizational knowledge?
  - Keeping software and platforms up to date?
- What are "key load parameters" for GitHub? Repos? Forks? Contributors?
  - "Whale" organizations like Shopify
  - Subscribers to repository
  - Contributors to repository
  - Fan out for notifications (size of community)
  - Actions that kick off a bunch of other git-heavy operations
  - Actions subscribing to every event (can overload Redis or webhook consumers)
    - Helped by moving away from RPC to message queues
    - Thinking of moving to eventing system (immutable event log)
- Does GitHub have a chaos team? YES

#### Team discussion:

- Logging is a forensic tool
- Tracing example: append request id to each new request in chain
- Celebrities as lever for scaling:
  - "What do you do when Beyonce joins your app?"
  - Twitter example: used to have a dedicated server for Justin Beiber