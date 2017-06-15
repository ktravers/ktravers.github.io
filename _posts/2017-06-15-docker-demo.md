---
layout: post
title: Code Reading - Learn ❤️ Docker
---

#### Prereqs:
  - Watch this [Youtube playlist](https://www.youtube.com/playlist?list=PLbG4OyfwIxjEe5Y3hQCiQjYnSgRH051iJ) (~90 min)
  - Clone down existing Rails app (in our case, the Learn codebase)
  - Set up account on [Docker Hub](https://hub.docker.com/)

## Intro

#### What is Docker?
  - Tool for managing and creating containers

#### Why use Docker?
  - Maintain a stable, consistent environment
  - Encapsulate all pices of an app
  - **Uniformity** and **consistency**
  - Step towards better deployments and auto-scaling infrastructure (Swarm, Kubernetes)
  - Lighter than a VM (no hypervisor)
  - One of the more mature tools available

## Basics

#### Docker image:
  - Pre-packaged layer
  - Can stack layers on each other
  - Result of running Docker file
  - Can create containers off of image

#### Docker container:
  - Running isolated container of your app's code

#### Docker networking:
  - Built-in
  - Allows containers to talk to each other over same network

#### Docker volumes:
  - Take local files and put them into your container
  - Edit locally, see changes in container

#### Docker compose:
  - Toolset for coordinating containers

#### [Docker Hub](https://hub.docker.com/):
  - Store and access repositories (images)

## Practice

Run an image: `docker run -it ubuntu` (creates container with interactive terminal)

Build Docker file: `docker build -t wget .` (Note: `.` means look for Docker file in current directory)

```
FROM ruby:2.3

RUN apt-get update -yqq \
  && apt-get install -yqq --no-install-recommends \
  postgresql-client

WORKDIR /usr/src/app
COPY Gemfile* ./
RUN bundle install
COPY . .

EXPOSE 3000
CMD rails server -b 0.0.0.0
```

Side note: `rails server -b 0.0.0.0` binds to all ports.

## Workshop: running Learn on Docker

1. Create Dockerfile in app directory (app === build context)
  ```
  FROM ruby:2.2.2cat

  RUN apt-get update -yqq \
    && apt-get install -yqq --no-install-recommends \
    postgresql-client nodejs npm \
    && npm install gulp -g \
    # symlink nodejs -> node
    && ln -s /usr/bin/nodejs /usr/bin/node

  WORKDIR /var/app

  COPY Gemfile* ./
  RUN bundle install

  COPY package.json ./
  RUN npm install

  COPY . .
  ```
2. Optional: add [.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
3. `cat .gitignore >> .dockerignore`
4. Build image: `docker build -t ironboard .`



## Additional resources

- Docker Docs: [https://docs.docker.com/compose/rails/](https://docs.docker.com/compose/rails/)
- [Docker for Rails Developers](https://medium.com/@charlie.b.ohara/docker-for-rails-developers-5a2a6c2c0593)
- [Dockerizing a Ruby-on-Rails Application](https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application)
- [How to Set Up a Private Docker Registry on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04)
