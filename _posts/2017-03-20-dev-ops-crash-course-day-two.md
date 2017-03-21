---
layout: post
title: Dev Ops Crash Course - Day Two
---

See notes from day one here: http://blog.kate-travers.com/dev-ops-crash-course-day-one/

## Year In Review Review

Q: What is DevOps?

A: When first created, more of a concept than a well-defined position. Even today, the term is still fairly broad, but in general refers to:

> ..a set of practices emphasizing the collaboration & communication of both
> software developers and operations professionals, while automating the process
> of software delivery and infrastructure changes.
> - [@devinburnette](https://github.com/devinburnette)

### Goal 1: Strengthen our system

Eliminate single points of failure. Affects revenue and trust.

One example of this: setting up Postgres Database Replication
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


## Debugging tools

### Get Your Bearings

Lookup current hostnames for db/environments: Digital Ocean > Networking > Floating IPs (or lookit Chef Nodes)

VIP: "virtual IP"
  - entry point into your balancing strategy
  - in our case, the floating IP

`/var` is usually log directory.  
`/etc` is usually config directory.

### Users and Permissions

**Users:**

Get all list of all users: `less /etc/passwd`  
Get list of all users in groups: `less /etc/groups`

Every user has an id. Groups have ids, too.

Every service that runs on server should have its own user. Reduces vulnerabilities.

We store dev team keys under single user to maintain separate concerns.

We also have a separate group for users with root access (all other users on box DO NOT have root access).

**Permissions:**

```
| Special     | User         | Group          | World          |
|:------------|:-------------|:---------------|:---------------|
| (d)ir       | (r)ead == 4  | (w)rite == 2   | e(x)ecute == 1 |
|             | rwx          | rwx            | rwx            |
|             | 101          | 001            | 111            |
|             | 5            | 1              | 7              |
```
