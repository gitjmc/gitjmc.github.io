---
title: Dockerize MySql
author:
  name: 
  link: 
date: 2022-05-20 21:20:00 -0400
categories: [Docker, Mysql]
tags: [dev, docker, tuto]
render_with_liquid: false
---

This post will guide you how to dockerize MySql.

## Prerequistes

 - Docker installed;
 - docker-compose.yaml
 
```yaml
version: '3'
services:
  db:
    image: mysql:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=root-password
      - MYSQL_DATABASE=databasename
      - MYSQL_USER=user
      - MYSQL_PASSWORD=user-password
#      - TZ=Europe/Paris
    volumes:
      - ~/datastore/mysql/dbdev:/var/lib/mysql
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 2012:3600
```
{: file='docker-compose.yml' .nolineno}


## Backup and Restore Data
### Backup
````console
$ docker exec -it <IMAGE NAME> /bin/bash

$ sudo docker exec -it db mysql -uroot -p

....

mysql> 

````

````console
mysql> show databases;
---
mysql> show tables;
---
mysql> use tableName;
---

## to restore database
mysql> create database dbName;

mysql> exit;
````

````console
$ mysqldump -u userName -p pwd --no-data dbName > backupFile.sql    (without data)
$ mysqldump -u userName -p pwd dbName > backupFile.sql
$ mysqldump -u userName -p pwd --all-databases > backupFile.sql
$ mysqldump -u root -p ******** --all-databases > backupFile.sql
````

## Restore

````console
$ mysql -u userName -p pwd dbName < backupFile.sql
````

> Update users password, otherwise you won't be able to connect from outside.
{: .prompt-danger }

Run these commands from the host (not in the docker container)

```console
$ SET PASSWORD FOR 'root'@'%' = 'p@ssw0rd';
$ SET PASSWORD FOR 'app_user'@'%' = 'p@ssw0rd';
```

## Learn More

For more knowledge about time in docker, visit the [5 ways to change time in docker container](https://bobcares.com/blog/change-time-in-docker-container/).
