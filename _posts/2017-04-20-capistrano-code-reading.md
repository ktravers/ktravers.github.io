---
layout: post
title: Code Reading - Capistrano
tags: ['code reading', 'capistrano', 'deployment']
---

A fascinating glimpse into [Spencer's](https://github.com/spencer1248) server config process.

### Impetus:

Contractor working on our marketing website CMS needs a staging environment to deploy changes to.

### Context:

We set up a one-click droplet on Digital Ocean. But we use chef for server management, so we need to custom configure this setup.

### Requirements:

- Wordpress
- MySQL
- [Varnish](https://varnish-cache.org/) (service that sits in front of web applications like Wordpress, handles caching, kinda like nginx with caching)

### What we did:

First step was looking at our existing chef cookbooks and seeing if there's anything we can reuse. Turns out we have `fis-msql` cookbook already - but why? We don't use MySQL on production. So look for a corresponding droplet on Digital Ocean; can't find one.

Git blamed cookbook, and it was added in March around the same time this marketing site CMS project was originally kicked off (later put on paused, restarted now).

_Side note:_ Spencer is using the Elflord color scheme for VIM. Loves it.

Diving into `fis-msql` cookbook code:  
  - see our base cookbook
  - setting up msql v5.6, so update to latest v5.7 (bc why not)
  - rm `mysql2_chef_gem` (don't need it)
  - rm [`chef-cookbooks/database`](https://github.com/chef-cookbooks/database) bc it was doomed and deprecated
  - rm all code associated with deprecated `database` cookbook

Lots of unknowns:  
  - Seeing attributes for `db_names`
  - Seeing references to [Scotch Box](https://box.scotch.io/), a Vagrant LAMP stack.
  - Where did `flatiron-v3-db.sql` come from? is it worth keeping?

Next tried setting up regular ol' [Vagrant](https://www.vagrantup.com/) (the best supported utility machine for Mac, de facto standard). Then learned that better to use vagrant-berkshelf thing. Then saw that maintainers actually favored using [`test-kitchen`](https://docs.chef.io/kitchen.html).

_Side note:_ we tried getting [Vagrant](https://www.vagrantup.com/) running with Ironbroker, RabbitMQ, etc., but turned out to be waaaay too big a project for us. Abandoned.

Benefits of [`test-kitchen`](https://docs.chef.io/kitchen.html):  
  - can run script locally to see how it builds
  - sets up virtual machine
  - ops code supported; good documentation

Ran into a problem: SSH keys are awkward because of the way our fis-users cookbook is set up. We kinda block root user, and default Vagrant user isn't in default admin list. Workaround: test-kitchen spins up an ssh key pair on the fly and uses that instead.

Now can run `kitchen login` to ssh onto box as vagrant user. BUT how do we ssh in as a different user? Set username in `.kitchen/default-ubuntu-1604.yml` (instance specific config).

Once we're on the box, turns out you can't `sudo mysql...` because mysql has its own permissions layer.

### Next steps:

1. Adding Varnish
2. Getting that sudo permissions stuff working
3. Once we've got this all working, maybe add a QA environment too?
