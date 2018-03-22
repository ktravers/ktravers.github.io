---
layout: post
title: Ops Roadmap for Learn.co
---

### Outline:

1. Review current setup
2. Share upcoming challenges/priorities
3. Share roadmap + new concepts/tools

#### Priorities:

1. Ensure new collaborative ventures are successful
2. Support our team as we grow (make ops more automated + manageable)

#### Requirements:

- Move to AWS for hosting
- Need high amounts of infrastructure + environment automation and orchestration ([Terraform](https://www.terraform.io/))
- Scaling
- Security
- Lower maintenance costs as our team grows

## Current Setup

- Hosted on [Digital Ocean](https://www.digitalocean.com/)
- Self-hosted services:
  - Postgres
  - Redis
  - Elastisearch
  - Memcached
  - Pushstream
- Our virtual servers are on private network in DO region

### Pain points

- Communication between services is not automated (no robust tooling available)
- Our servers are "pets not cattle"
- High maintenance costs
  - Lots of outages
  - Infrastructure is not self-healing (no robust tooling available)
- Low security
- Noisy alerts (Nagios)
- Relying on manageable (aka more brittle) deployment and provisioning processes (Chef)
- Our virtual servers are on shared machines, so vulnerable to leaks / attacks

## Roadmap

### Security

Guiding principle: Principle of Least Privilege (limit surface area / attack vectors)

- [x] GitHub 2FA
- [x] Remove root AWS keys
- [ ] Secrets encryption through [AWS KMS](https://aws.amazon.com/kms/) (in progress)
- [ ] VPN for local development
- [ ] Migrate to [AWS Virtual Private Cloud](https://aws.amazon.com/vpc/)

#### More on [AWS Virtual Private Cloud](https://aws.amazon.com/vpc/)

- Public and Private subnets
- Services that don't need to be exposed to internet (redis, etc.) will live in private subnet
- NAT Gateway rules to manage traffic

### Scaling

All about automation

- [ ] Managed services instead of self-hosting
- [x] Migrate DNS from Dyn to [AWS Route 53](https://aws.amazon.com/route53/)
- [ ] [Terraform](https://www.terraform.io/) for "Infrastructure as Code" orchestration automation
- [ ] [Packer](https://www.packer.io/docs/builders/amazon-ebs.html) for automated AMI builds (images for Amazon instances)

- Additional things we're thinking about:
  - Deployments
  - Alerting
  - Monitoring
  - Logs

#### More about [Terraform](https://www.terraform.io/)

Infrastructure as code: automates your environment to match your config file (declarative code)

- Source controlled code
- Reduces documentation (self-documenting system)
- Support for multiple cloud providers

### More in the Learn.co ops series

- [Ops Crash Course: Day One](http://blog.kate-travers.com/dev-ops-crash-course-day-one/)
- [Ops Crash Course: Day Two](http://blog.kate-travers.com/dev-ops-crash-course-day-two/)
- [Ops Crash Course: Day Three](http://blog.kate-travers.com/dev-ops-crash-course-day-three/)
- [Ops Crash Course: Day Four](http://blog.kate-travers.com/dev-ops-crash-course-day-four/)
- [Ops Crash Course: Day Five](http://blog.kate-travers.com/dev-ops-crash-course-day-five/)
