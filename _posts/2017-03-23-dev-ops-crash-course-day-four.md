---
layout: post
title: Dev Ops Crash Course - Day Four
---

Notes from [day one](http://blog.kate-travers.com/dev-ops-crash-course-day-one/), [day two](http://blog.kate-travers.com/dev-ops-crash-course-day-two/) and [day three](http://blog.kate-travers.com/dev-ops-crash-course-day-three/).

## Learn IDE

### DNS

In general, SSL certificate needs to match what IP address the web request resolves to (resolves to DNS entry). DNS entry is for floating IP. So you can use the same certificate for multiple servers, depending on what server floating IP points to. That allows us to SSL terminate at load balancer server. Certicate is on load balancer.

We have load balancers in front of each region AND a super load balancer to handle delegation. Has 3 different A records, one per load balancer.

We're using traffic management service on Dynect. This service allows you to set address pools and service controls.

#### Address Pools

Pools are preset by Dynect. We plugged in our servers where it made sense.

Global pool has all 3 main regional load balancers. Monitor and obey are active.

Serve mode off for US Central, so people get routed to closest region.


#### Regional rules

Set auto recover, which server to failover to.


#### Service Controls

TTL setting.

Health monitoring - originally handled load balancing, now just checking if balancers are up.


### Chef

We have a separate [chef domain](chef.students.learn.co) and [repo](https://github.com/flatiron-labs/students-chef-repo) for IDE infrastructure.

When you build a new release, it doesn't get built on your machine. It gets built on elixir-build server, then distributed.


#### Cookbook

Using haproxy for load balancers. Config lives in `/usr/local/etc/haproxy`

IDE config is using slightly different approach than Learn. Not using cookie. Instead using hashed token from user (as url param).


#### TODOs

- Currently we have more servers than we need running this elixir application. We should/can scale back. Gluster servers can _probably_ be deprecated.
- haproxy default recipe doesn't seem to be up-to-date. Ask Devin to push file that may or may not be on his local.
- Down the road: use simpler Elixir load balancer where we have more control over hashing algorithm.


### Monitoring

[Wombat](wombat01.students.learn.co:8080)
