---
layout: post
title: Dev Ops Crash Course - Day Five
tags: [code reading, devops, chef]
---

Notes from [day one]({% post_url 2017-03-20-dev-ops-crash-course-day-one %}), [day two]({% post_url 2017-03-21-dev-ops-crash-course-day-two %}), [day three]({% post_url 2017-03-22-dev-ops-crash-course-day-three %}), and [day four]({% post_url 2017-03-23-dev-ops-crash-course-day-four %}).

## Chef Chat

### What is Chef?

- Scripts written in Ruby to provision a server.
- Server setup
- Infrastructure automation
- Server configuration management

### What's important to manage?

- Software versioning
- Uniformity and consistency across machines
- Reproducability
- Idempotency

### What problems are we solving?

- Efficient setup
- Preserves history
- Prevents environment issues resulting from inconsistencies among envs

### Alternatives to Chef?

- Server image
- Shared shell script on GitHub
- [Ansible](https://www.ansible.com/)
- [Puppet](https://puppet.com/)

### How We've Structured Our Cookbooks

Rely on inheritance. All cookbooks include base cookbook, which makes sure we always have security standards, base stuff in place on all boxes.

[Wrapper cookbooks](https://blog.chef.io/2013/12/03/doing-wrapper-cookbooks-right/) let us have a general template we can extend with app-specific details.

### Idiosyncracies of Chef

Conventions aren't very clearly defined. Easy to define attributes all over the place (ex. in your recipes, in attributes directory, etc.)

Cookbooks have many recipes, but in our chef repo, we tend to only have one recipe per cookbook (the default recipe).

### Community cookbooks

When using community cookbooks, probably want to wrap rather than fork. After a certain amount of customization, merits writing your own. We've had some work out w/o any customization (Rabbit), but others we started from then ended up writing our own (Ruby cookbook).

Plus it's hard to use a community cookbook if you don't understand server setup to begin with, so worth it to gain knowledge by writing your own.

Another good exercise: set up servers by hand (like we did on [day one]({% post_url 2017-03-20-dev-ops-crash-course-day-one %}).

### Testing cookbooks

[ChefSpec](https://sethvargo.github.io/chefspec/)

[ServerSpec](http://serverspec.org/)
