---
title: Install Docker on Deepin 20.x
author: 
  name: 
  link:
date: 2022-02-27 15:03:00 -0500
categories: [Linux]
tags: [deepin, docker, os]
math: true
mermaid: true
image:
  src: /commons/devices-mockup.png
  width: 400
  height: 250
---

Cet article montre comment peut-on installer docker dans Deepin[^ref1] version 20.x et plus.

> MAJ '2022-04-18'
{: .prompt-tip }


# Solution
---
## install Docker

### Prerequistes
````console
$ sudo apt-get update
$ sudo apt install -y apt-transport-https ca-certificates gnupg2 curl software-properties-common
$ sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
````
#### Check source.list file
path `/etc/apt/sources.list`{: .filepath}
path `/etc/apt/sources.list.d/docker-ce.list`{: .filepath}
````console
$ cat /etc/apt/sources.list
````
#### Install docker
````console
$ sudo apt-get update
$ sudo apt-get install -y docker-ce
$ docker --version
````
## install docker-machine
Source pour installer docker machine [^ref2].
````console
$ curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
>    chmod +x /tmp/docker-machine &&
>    sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
$ docker-machine --version
````

## install docker-compose
Source pour installer docker compose[^ref3].
````console
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose version
````

----

voir aussi cette source[^ref4] 

`deb [arch=amd64] https://download.docker.com/linux/debian buster stable`


```diff
deb [by-hash=force] https://community-packages.deepin.com/deepin/ apricot main contrib non-free
deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian buster contrib
+ deb [arch=amd64] https://download.docker.com/linux/debian buster stable
```
{: .nolineno file="sources.list" }

> `buster` - est la version Debian de Deepin
{: .prompt-info }

---

### Reverse Footnote

[^ref1]: [Voir ce liensur youtube](https://www.youtube.com/watch?v=WoHffLF0jfU)
[^ref2]: [source in github docker machine](https://github.com/docker/machine/releases/tag/v0.16.2)
[^ref3]: [source in github docker compose](https://docs.docker.com/compose/install/)
[^ref4]: [source article](https://medium.com/@leonardo.lp2/installing-docker-on-deepin-15-9-2312f28f09b7)

 
