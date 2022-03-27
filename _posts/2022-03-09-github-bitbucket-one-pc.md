---
title: manage github and bitbucket accounts on the same computer
author:
  name: 
  link: 
date: 2022-03-06 08:46:00 -0500
categories: [cloud, repository]
tags: [dev, tuto, github, bitbucket]
---

How to manage multiple accounts github and bitbucket on the same computer ...\o/...

# Recette

## Create config file in .ssh folder

```yaml
#Default account
Host bibucket.org
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/id_rsa

#Personal account
Host bitbucket-perso
  Hostname bitbucket.org
  User git
  IdentityFile ~/.ssh/id_rsa_perso

#professsional account
Host bitbucket-mtl
  Hostname bitbucket.org
  User git
  IdentityFile ~/.ssh/id_rsa_mtl

#github account - not created yet!
#Host github.com
#  Hostname github.com
#  User git
#  IdentityFile ~/.ssh/id_

#gitjmc account
Host gitjmc
  Hostname github.com
  User git
  IdentityFile ~/.ssh/id_ed25519

#gitmohamedcjdev account
Host gitmoh
  Hostname github.com
  User git
  IdentityFile ~/.ssh/gitmoh
```
{: file='.ssh/config' .nolineno}

```console
$ bundle
```
