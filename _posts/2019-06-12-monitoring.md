---
layout: post
title: Modern Monitoring and Observability with Thoughtful Chaos Experiments by Datadog and Gremlin (webinar)
tags: ['training', 'monitoring', 'observability']
---

Notes from "Chaos engineering" webinar with Datadog and Gremlin.

Presenters:

- Ana Medina
  - Chaos engineer @ Gremlin ("Chaos engineering as a service")
  - @ana_m_medina
- Jason Yee
  - Senior tech evangelist @ Datadog
  - @gitbisect

## Three Types of Data

### 1. Work metrics (Business)

- Throughput
- Success
- Performance
  - Latency
  - Perceived performance

### 2. Resource Metrics (Services)

- Utilization
- Saturation
- Availability

### 3. Events

Things that influence how our system(s) behave

- Code changes
- Scaling events
- CHAOS

## Monitoring Tools

### 1. Logs

- Information about an event
- Snapshot, but not aggregate

### 2. Metrics

- Context around an event
- Helps us see trends

### 3. Traces

- Causes leading to an event

## Example

https://github.com/jyee/guestbook

## Chaos Engineering

Evaluate your monitoring by running chaos experiments.

- Start in a dev or staging environment
- "Contain the blast radius" - keep the experiment safely scoped
- Start with a small experiment
  - Form hypothesis in advance
  - Test
  - Re-evaluate

## Datadog + Gremlin Integration

dtdg.co/gremlin-datadog

