---
layout: post
title: Dev Ops Crash Course - Day Three
---

Notes from [day one](http://blog.kate-travers.com/dev-ops-crash-course-day-one/) and [day two](http://blog.kate-travers.com/dev-ops-crash-course-day-two/).

## Year in Review Review (continued)

### Notifications

Nagios repo w/ full documentation: https://github.com/flatiron-labs/nagios

Nagios API that powers our Learn.co [status page](status.learn.co): https://github.com/zorkian/nagios-api

[Status page](status.learn.co) lives on nagios01 server: `root@nagios01:/usr/local/nagios/nagios-api`. There's a cron job that runs updates: `crontab -l` to view, `crontab -e` to edit.

### Automated SSH Key Propagation

Cron job that runs every 30min that runs chef `user_setup` recipe on every host.

Every 30min too often? We should scale this back.

### Automated Server Provisioning Tools

Script that uses Digital Ocean, Dynect, and Chef APIs to provision the host automatically. New host in under 2 min.

Note: script is a little dated now. DO has since made its own CLI tools that do a lot of this ([doctl](https://www.digitalocean.com/community/tutorials/how-to-use-doctl-the-official-digitalocean-command-line-client)). There's also [tugboat gem](https://github.com/petems/tugboat).

DO tools probably better suited since they incorporate all DO's latest features.

### Faster Deploys

[Drew](https://github.com/drewprice) and [Devin](https://github.com/devinburnette) [put work in](https://github.com/flatiron-labs/ironboard/pull/9920/files) to speed up server-side only deploys. More work needs to done on speeding up asset compilation.

### Weekly DB Migrations

See `qa-support` server. Runs cron job: `root@qa-support /etc/cron.d`. Why `cron.d`? Convention Devin likes to use when running cron jobs that need to be accessible to users but run by different user (for example, postgres user).

Takes daily db dump, terminates all qa db connections, drops + restores db from dump.

See full docs on [operations wiki](https://github.com/flatiron-labs/operations/wiki/Restoring-Production-db-to-QA).
