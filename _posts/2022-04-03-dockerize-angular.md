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

  - Pour ne pas installer sur son poste node, angular etc ...
  - Aussi avoir la possibilt/ de travailler avec differentes versions de `node` et/ou `angular`

## Comment

en suivant [**cette source**](https://medium.com/@tal.ohana.x/dockerizing-angular-development-environment-done-right-4527155578fa) 

1. cloner le repos (sinon cr/ere un nouveau avec son seed)
2. verifier `Dockerfile` et `docker-compose.yml`
  - workdir
  - la version de node
  - le nom du service
  - le nom du container 
3. builder en executant `docker-compose up -d`
  - To rebuild this image you must use `docker-compose build` or `docker-compose up --build`
  
```console
$ docker-compose -f docker-compose.seed.yml run seed
# pour voir les logs
$ docker-compose logs -f dev
# pour acceder au container
$ docker-compose exec dev sh
```
4. to deploy to ghp 
  - in container add ssh key `ssh-keygen -t ed25519 -C "jmc@fromdocker.com"`
  - if not installed so add ghp `npm install -g ghp` or `ng add angular-cli-ghpages`
  - after each npm install repeat 5.


5. 
to fix errors on vscode, due to missed directory node_modules
run this batch
```console
$ docker cp {containerName}:/angapp/package-lock.json . && \
> docker cp {containerName}:/angapp/node_modules - > node_modules.tar && \
> tar -xf node_modules.tar && \
> rm -f node_modules.tar
```

when we generate components, services ... files in container, these are automaticly generated in the host.
So we have to change permission owner
```console
$ sudo chown -R jmc:jmc folderName/
```

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

## Errors

> node:internal/modules/cjs/loader:**936** throw err ...
{: .prompt-danger }

1. In container remove `node_modules`
2. run command `npm i` to create all dependicies ...
3. Add ssh if doesn't exist
4. re-run the batch to fix errors in vscode

## some commands cli

```console
# Check angular version
$ ng --version
# Check npm version
$ npm -v
```
