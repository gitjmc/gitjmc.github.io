---
title: Dockerize nextCloud
author:
  name: 
  link: 
date: 2023-02-16 20:55:00 -0400
categories: [Privacy]
tags: [nextcloud, nginx-proxymanager, tuto]
render_with_liquid: false
---

This post will guide you how to install nextcloud.
Je dois ajouter tous cce que je pourrai trouver ...
## Prerequistes

 - server;
 - docker;
 - nginx proxy manager;
 - ajouter les liens des la documentation ...
 
```yaml
version: '3'
services:
  jekyll-serve:
    image: jekyll/jekyll:4.0
    volumes: 
      - '.:/srv/jekyll'
    ports:
      - 4000:4000
    command: 'jekyll serve'
```
{: file='docker-compose.yml' .nolineno}

to run docker-compose
```console
$ docker-compose up
```


## Errors

> Unable to monitor directories for changes because iNotify max watches exceeded.
{: .prompt-danger }

Run these commands from the host (not in the docker container)
```console
$ echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p
```
