---
title: Dockerize Jekyll
author:
  name: 
  link: 
date: 2022-04-03 18:40:00 -0400
categories: [Docker, Jekyll]
tags: [dev, docker, tuto]
render_with_liquid: false
---

This post will guide you how to dockerize Jekyll.

## Prerequistes

 - Docker installed;
 - docker-compose.yaml
 
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


## Errors

> Unable to monitor directories for changes because iNotify max watches exceeded.
{: .prompt-danger }

Run these commands from the host (not in the docker container)
```console
$ echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p
```

## Learn More

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).
