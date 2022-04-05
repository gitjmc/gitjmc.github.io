---
title: Dockerize an Angular dev environment
author:
  name: 
  link: 
date: 2022-04-03 23:05:00 -0400
categories: [Docker, Angular]
tags: [dev, docker, tuto, angular]
---

This post will guide you how to create an angular dev environment in docker.

## Contexte

  - Pour ne installer sur son poste node, angular etc
  - Si on veut travailler avec differentes versions de `node` et/ou angular `angular`

## Comment

en suivant [**cette source**](https://medium.com/@tal.ohana.x/dockerizing-angular-development-environment-done-right-4527155578fa) 

1. cloner le repos (sinon cr/ere un nouveau avec son seed)
2. verifier dockerfile et docker compose puis builder
$ docker-compose -f docker-compose.seed.yml run seed
pour voir les logs
$ docker-compose logs -f dev
pour acceder au container
docker-compose exec dev sh

Pour ne pas avoir les erreure dasn vsCode, apres chaque npm install on doit rouler ce batch
docker cp dev:/angapp/package-lock.json . && \
docker cp dev:/angapp/node_modules - > node_modules.tar && \
    tar -xf node_modules.tar && \
    rm -f temp_node_modules.tar

quand on gener des composants, services ...
on doit changer les permission du repertoire et ses fichiers de root a jmc (dans mon cas) sinon a chaque modification il va demander la permission
$ sudo chown -R jmc:jmc folderName/
---
lancer le tout pour continuer le developpement:
lancer l'api: aller dans le rep de l'api 
$docker-compose up -d
lancer l'app-ang: aller dans le rep de l'app-ang
$docker-compose up -d
$docker-compose logs -f dev

pour le deploiement dans ghp
s'assurer que le package ghp est installe sinon $ng add angular-cli-ghpages
installer .ssh si c'est pas fait
$npm run deploy:gh

````console
🚀 Uploading via git, please wait...
🌟 Successfully published via angular-cli-ghpages! Have a nice day!
````

````console
\{^_^}/ hi!
````

## How?


## nodejs in Docker
