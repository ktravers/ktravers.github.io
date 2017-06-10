---
layout: post
title: Dev Ops Crash Course - Day Four
---

Notes from [day one](http://blog.kate-travers.com/dev-ops-crash-course-day-one/), [day two](http://blog.kate-travers.com/dev-ops-crash-course-day-two/) and [day three](http://blog.kate-travers.com/dev-ops-crash-course-day-three/).


## Learn IDE

### Load Balancers

Using haproxy for load balancers. Config lives in `/usr/local/etc/haproxy`

IDE config is using slightly different approach than Learn. Not using cookie. Instead using hashed token from user (as url param).


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

[Gluster](https://www.gluster.org/) manages distributing archives across servers. Gluster servers mount disk on archive server.

Gluster mounts onto `/archive/user_files` directory.


#### Container mgmt

In the user's home directory in the container, we're mounting directory from host machine (similar to network drive mounted on your local computer). Things get written to hard disk while using it, but only in mounted directory. See [union file system](https://en.wikipedia.org/wiki/Aufs) and [UnionFS](https://en.wikipedia.org/wiki/OverlayFS).

We use [Rocket](https://github.com/coreos/rkt) instead of [Docker](https://www.docker.com/). Why?  
  - Docker slow closing down
  - Docker uses daemon to run all container creation commands through, and it doesn't handle heavy load well
  - When under heavy load, it'll tell you container started, but it's actually NOT

[runpty](https://en.wikipedia.org/wiki/Pseudoterminal) - without using this, we can't properly stream output from terminal stream (rkt container) to Elixir port. Does something with rkt command to make it work.


#### Cron

When you run a cron job, it sends mail to user. Run `mail` command to see messages.


#### Templates

`vm.args.erb`: lives on server so when build process finishes, copies args into the right place. Essentially a config file.


#### Files

Bash scripts used by Rocket. Hijacks setup scripts. Tells it to install in container.


#### Build script

Take out `acbuild --debug run -- /bin/bash -l -c "easy_install tzupdate nose nose-json"`? For python, which we don't use.

`create_user` script:  
  - when container gets spun up, pass in a variable so we can create user account for the actual connected user.
  - `chown`s permissions


#### TODOs

- Currently we have more servers than we need running this [Elixir](http://elixir-lang.org/docs.html) application. We should/can scale back. Gluster servers can _probably_ be deprecated.
- haproxy default recipe doesn't seem to be up-to-date. Ask Devin to push file that may or may not be on his local.
- Down the road: use simpler Elixir load balancer where we have more control over hashing algorithm.


### IDE Backend

`ide_umbrella`: main application for IDE backend. Elixir app.

It's not actually an umbrella application (aka an app that manages a bunch of microservices). Name is a misnomer. Originally conceived as a true umbrella app, but then took different approach and never renamed.

`.deliver` directory: builds releases, orchestrates delivering to server. Puts abstraction on top of [distillery](https://github.com/bitwalker/distillery).

#### Dependencies

Listed in mix.exs file. Run `mix deps.get`

- [`gproc`](https://github.com/uwiger/gproc): global process registry
- [`distillery`](https://github.com/bitwalker/distillery): distribution
- [`poolboy`](https://github.com/devinus/poolboy): pooling system
- [`cowboy`](https://github.com/ninenines/cowboy): web server


#### Running the app

Comment out any rkt stuff, since it's a pain to get that running locally (`lib/ide/remote_server/supervisor.ex`)

Jump into interactive console: `iex -S mix`

Observer: `:observer.start`. Hard to run on servers, but could install something like `wobserver` for this instead. [Logan](https://github.com/loganhasson) has a branch up for this; working, but needs auth on endpoint.

GenServers: any time you need to manage state plus a little behavior, use a GenServer. Similar to when you'd use a Ruby object.


#### Debugging:

```erlang
require IEx
IEx.pry
```

### Monitoring

[Wombat](wombat01.students.learn.co:8080)


### Deployments

Can deploy a release (not hot swap) or an upgrade (hot swap).

Upgrade ships to server w/ relup file that contains set of instructions for running hot swap.

Ideally, we want to do upgrade (hot swap) whenever possible.

Today:  
  1. Build release and deploy to QA
  2. Run upgrade on QA
  3. Run upgrade on prod

Deploy config in `.deliver` directory.

For upgrade to work, version names need to be sequential.

From master:  
  1. Check version: `mix edeliver version qa`
  2. Build release: `mix edeliver build release`
  3. Deploy release to QA: `mix edeliver deploy release to qa` then enter version (or run `mix edeliver deploy release to qa --version=x`)
  4. `mix edeliver restart qa` but there's something wrong with our QA server, where this command doesn't actually restart the server.
  5. Manually restart server after ssh'ing into QA: `ssh ide-qa-server`, become deployer, `cd ide`, `bin/ide start`, `bin/ide attach`, then ctrl+d to exit
  6. Build upgrade: `mix edeliver build upgrade --with=x` where x is version on production
  7. Deploy upgrade to QA: `mix edeliver deploy upgrade to qa`
  8. Check version: `mix edeliver version qa`
  9. Deploy upgrade to production: `mix edeliver deploy upgrade to production`
  10. Check version: `mix edeliver version production`


### Mini Elixir App Demo

1. `mix new seltzer`
2. Entry point: `lib/seltzer.ex`
3. Run tests `mix test`
4. Recompile `r(Seltzer)` (name of module)
