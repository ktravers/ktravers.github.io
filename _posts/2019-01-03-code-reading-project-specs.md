---
layout: post
title: Code Reading - How Write To Project Specs
tags: ['code reading']
---

Learnings from @bobjnadler's time at Siemens & Cyrus.

- Helpful to have templates
- Remember: questions > formatting

## What is a spec?

Q: Technical spec? User stories? Diagrams?

A: Combination of several pieces of information, only some of which is technical.

### Parts

#### Charter

- Overview
- Biz case
- Goals
- Requirements
- Assumptions
- Resources
- Constraints
- Risks
- Decision makers: stakeholders, sponsor(s), manager(s)

Useful as an engagement offer, aka proposal for client

Project matrix:  
- Drivers for decision making
- Cost vs. release date vs. feature set

##### Guiding Questions

- What is the problem we're trying to solve?
- Who is this problem affecting? (scale, scope)
- How is it affecting them? (impact)
- What are known issues external to project that can affect drivers (resources, features, scope)?

#### Vision

- More details on vision, customers, etc
- High level features
- Biz value analysis

##### Guiding Questions

- How does this connect to biz goals?
- Why is this project more relevant to others in terms of achieving those goals?
- What evidence do we have that this will solve customers' problems?
- How will customers get work done without this project?
- Why will customers use this project?
- Who are the competitors and how will this project compare?
- What solutions have been requested/suggested and why aren't we doing that instead?
- What's out of scope for this project?

#### Spec

- Stories with supporting info like:
  - UI mockups
  - Domain models
  - Activity models
  - Decision tables
  - Glossary

##### Guiding Questions

- What ubiquitous language will the project use?
- What technologies are currently in use? Are we bound to them?

#### Plan

- Lifecycle, phases, approach (XP, scrum, etc)
- Ground rules
- Initial estimates
- Plan for reporting

Example: project Summit where representative from all involved teams get together for a status update.

##### Guiding Questions

- How will issues be tracked and fixed?
- How will story completion be tracked?
- What methods/strategies are used for estimating effort? timeline?
- Estimates should be refined at regular intervals. What are those intervals?
- Who do we need to report to?
- What should those reports contain?
- Testing plan?
- Deployment plan?
- Launch plan?
- Education/training plan?

#### Risks

- Anything that could impact success

##### Guiding Questions

- What are likely ways this project will fail? (and how to avoid them)
- What/who is this project depending on to succeed?
- What/who is depending on this project to succeed?
- What assumptions are being made that this project depends on?
