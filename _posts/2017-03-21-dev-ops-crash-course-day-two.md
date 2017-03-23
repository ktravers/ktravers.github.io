---
layout: post
title: Dev Ops Crash Course - Day Two
---

Notes from [day one](http://blog.kate-travers.com/dev-ops-crash-course-day-one/).

## Year In Review Review

Q: What is DevOps?

A: When first created, more of a concept than a well-defined position. Even today, the term is still fairly broad, but in general refers to:

> ..a set of practices emphasizing the collaboration & communication of both
> software developers and operations professionals, while automating the process
> of software delivery and infrastructure changes.
> - [@devinburnette](https://github.com/devinburnette)

### [Postgres Database Replication](https://github.com/flatiron-labs/operations/wiki/PostgreSQL-Replication)

- Read-Only replica: all transactions are mirrored to 2nd server
- Hot standby in case of failure
- Easy failover strategy
- Lowers risk of downtime

Backups stored in `/var/lib/postgres` along with backup config.

#### How restore works:

- [WAL-E](https://github.com/wal-e/wal-e) (continuous archiving) takes base image of our database once a day and stores in S3.
- Postgres creates binary transaction logs (write-ahead logs) that have to complete before transaction succeeds. That way, can re-run if database crashes before completing transaction. Over the course of the day, we send up these write-ahead log files to S3.
- For restore, first imports base image to db, then applies deltas (write-ahead logs).
- WAL-E has interface for deleting backups. We go in every 6mo or so and rms old files.

Recovery config `/var/lib/postgres/9.3/main/recovery.conf` lives on replica. When designated trigger file is created on the replica, then it becomes primary and starts running recovery.

Kick this off by running script (see operations wiki article).

#### Future work:

- Set up another stand-by replica
- Use something like [Octopus gem](https://github.com/thiagopradi/octopus) to handle [replication](https://github.com/thiagopradi/octopus/wiki/replication), configured through yaml file

### Load Balancers

Load balancer in front of all of our hosts. Default in front of Learn. Separate ones for Rabbit, [Elasticsearch](https://github.com/elastic/elasticsearch), etc.

We get automatic failover from [keepalived](https://supermarket.chef.io/cookbooks/keepalived)

When healthcheck fails, keepalived runs master script that reassigns the floating IP.

See operations wiki for more info on [HAProxy Automatic Failover](https://github.com/flatiron-labs/operations/wiki/HAProxy-Automatic-Failover).

#### To inspect:

1. ssh into load balancer (see DO floating ips for what's active)
2. config is in `/user/local/etc/haproxy`
3. config set by chef recipe

#### Config details

- `listen admin` creates an admin portal (but not accessible now bc bound locally`)
- `frontend` where initial connection happens (i.e. learn.co), where we terminate ssl
- `backend` our actual web servers

haproxy backend makes a httpchk GET request to `*` that hits a healthcheck route defined in Rails app `routes.rb`. Which brings us to...

### Healthchecks

When [Elasticsearch](https://github.com/elastic/elasticsearch) is down, healthchecks will fail, and that server will be pulled out of pool. Temporary fix was to use load balancer to fail over to another Elasticsearch server, but issue remains if all Elasticsearch servers + load balancer are taken down (for example, during DO maintenance on 3/24).

Proposed solution: pull [Elasticsearch](https://github.com/elastic/elasticsearch) out of healthchecks.

See `Healthchecks` initializer, model, controller.


## Debugging tools

### Get Your Bearings

Lookup current hostnames for db/environments: Digital Ocean > Networking > Floating IPs (or lookit Chef Nodes)

VIP: "virtual IP"
  - entry point into your balancing strategy
  - in our case, the floating IP

`/var` is usually log directory.  
`/etc` is usually config directory.

### Users and Permissions

#### Users

Get all list of all users: `less /etc/passwd`  
Get list of all users in groups: `less /etc/groups`

Every user has an id. Groups have ids, too.

Every service that runs on server should have its own user. Reduces vulnerabilities.

We store dev team keys under single user to maintain separate concerns.

We also have a separate group for users with root access (all other users on box DO NOT have root access).

#### Permissions

```
| Special     | User         | Group          | World          |
|:------------|:-------------|:---------------|:---------------|
| (d)ir       | (r)ead == 4  | (w)rite == 2   | e(x)ecute == 1 |
|             | rwx          | rwx            | rwx            |
|             | 101          | 001            | 111            |
|             | 5            | 1              | 7              |
```

### Live Debugging: Server Goes Down

1. Gather info
 - Check memory on Librato
 - Run `htop` on server
 - Check logs on server (`root@/var/logs/apache2/error.log`)
 - Lookit [Passenger](https://www.phusionpassenger.com/library/walkthroughs/basics/nodejs/) processes: `sudo passenger-status --show=requests`
2. In this case (3/21), requests suggest that problem might be with Elasticsearch (showing a lot of search uris)
  ```
  root@ironboard08:/var/log/apache2# rvmsudo passenger-status --show=requests | grep uri
      uri                         = <search endpoint>
      uri                         = <search endpoint>
      uri                         = <search endpoint>
      uri                         = <search endpoing>
      uri                         = <search endpoing>
      uri                         = <search endpoing>
      uri                         = <search endpoing>
      uri                         = <search endpoing>
      uri                         = <search endpoing>
      uri                         = <search endpoing>
      uri                         = /api/v1/users/me?ile_login=true
      uri                         = *
      uri                         = *
  ```
3. Hypothesis: do we have a timeout on Elasticsearch? Are we DDOSing ourselves?
4. Test hypothesis: hit endpoint from browser
5. Confirmed: DDOSing endpoint brought down Learn.co
6. Restart all servers to bring back up

Vulnerabilities identified:
  1. Healthcheck: when [Elasticsearch](https://github.com/elastic/elasticsearch) is down, healthcheck fails and load balancer takes all servers out of rotation, 500ing the site
  2. [Searchkick](https://github.com/ankane/searchkick): when Elasticsearch is down, Searchkick indexing fails and 500s the site
  3. [Passenger](https://www.phusionpassenger.com/library/walkthroughs/basics/nodejs/): long search requests don't timeout, overload queue

Remediation:
  1. Decouple Elasticsearch from Learn (bringing down Elasticsearch should not bring down site)
  2. Add timeouts to Elasticsearch
  3. Throttle Elasticsearch requests from client- and server-side (so we're not sending 1 character queries)
  4. Upgrade Elasticsearch version (?)
