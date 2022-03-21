---
title: Multiple accounts (github and/or bitbucket) on the same computer
author:
  name: 
  link: 
date: 2022-03-06 08:46:00 -0500
categories: [Internet, repository]
tags: [dev, tuto, github, bitbucket]
---

How to work with multiple accounts of github and bitbucket on the same computer ...\o/...

Cette page contient des liens que j'ai rencontré qq part de partout ...  

# Liste des liens

## Ouitl de documentation en ligne

1. json editor - <https://jsoneditoronline.org/>
2. sequence diagrams - <https://www.websequencediagrams.com/>
3. sequence diagrams js - <https://bramp.github.io/js-sequence-diagrams/>
4. diagrams online - <https://draw.io>
5. Samrt Arabic Keyboard - <http://www.yamli.com/arabic-keyboard/>

## DEV
### front end (client-side)
- Bootstrap:
  - Bootstrap Icons - ­<https://icons.getbootstrap.com/#usage>
  - Doc Bootstrap 4 - <https://getbootstrap.com/docs/4.0/components/>
### CSS
  - css stats - Analyser et visualiser le stylesheets <https://cssstats.com/>
  - PageSpeed Insights <https://pagespeed.web.dev/>
  - Web Developer Checklist <https://www.toptal.com/developers/webdevchecklist>
### RegEx
  - RegEx Testing - <https://www.regextester.com/21>
  - RegEx Java Tuto - <http://www.vogella.com/tutorials/JavaRegularExpressions/article.html>
  - online tester - 
### free Photos
  - Free stock Photo - <https://www.pexels.com/>
### Generator online
- hipster - <https://hipsum.co/>
- Genrate data - <http://www.generatedata.com/>
- Random generatot - <https://www.4devs.com.br/>
### https
- certbot - <https://certbot.eff.org/>
- ssl config generator - <https://mozilla.github.io/server-side-tls/ssl-config-generator/>
- Let's encrypt - <https://letsencrypt.org/>

## DevOps
### Docker
  - using volumes - <https://devopsheaven.com/docker/docker-compose/volumes/2018/01/16/volumes-in-docker-compose.html>

## Autres
### Divers
  - mes noeuds - <http://mesnoeuds.free.fr/knot.php?knotId=31>
### Reflexion
  - Stratégie masquée de la guerre:
    - youtube - <https://www.youtube.com/watch?v=uz4NaV50wh0>
    - vimeo - <https://vimeo.com/129485806>
  - Journaliste achetés - youtube - <https://vimeo.com/129485806>

### 




email - nabi jerder - githubjn@gmail.com

github - githubjn

username - githubnj / githubjn
token - to regenrate if need

pour le d/ploiement
	angular-cli-ghpages

youtube - https://www.youtube.com/watch?v=8Mle8owZ3_4


voir les tuto de ssh 
	- voir OpenSSH Full guide - https://www.youtube.com/watch?v=YS5Zh7KExvE
	- voir Getting Started with OpenSSH Key Management - 
	- to create sshkey
		ssh-keygen -t ed25519 -C "jmc-dell"
	- to associate the key with target server or service you have to 
		ssh-copy-id -i ~/.ssh/nameOfKey.pub serverName


check if openssh is installed (always installed on linux), which command tell where the binary is installed
	which ssh
	apt search openssh-client

? use nano

check a specific process 
ps aux | grep process-id


the first time whene you connect to server, the server ask you if you are sure to ...
whene you answer yes, the fingerprint done by the server is stored in known_hosts

le repertoire des logs
	/var/log
pour suivre life le log des authentifications par exemple
	tail -f auth.log

ssh config file
---
Host nodejmc #peut etre n'importe quel nom
	Hostname 172.105.4.22 #le serveur qu'on veut acceder
	Port 22 
	User root  
Host nodenader
	Hostname 172.105.4.140
	Port 22 
	User jmc

----

le public key doit etre mis dans le serveur qu'on veut acceder
il doit etre dans le fichier (a creer s'ils n'existent pas)
	.ssh/authorized_keys 
la meme chose en utilisant 
	ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.105.7.26
id_rsa.pub - le public key qu'on veut copier
172.105.7.26 - le serveur qu'on veut y acceder

------------
ssh-i ~/.ssh/privateKey root@172.105.7.22
ssh agent - it's to save the key in memory to don't set each time you connect to server

to find ssh-agent
ps aux |grep ssh-agent

to activate it 
eval "$(ssh-agent)"

###
jmc@jmc-PC:/etc/apt$ ps aux |grep ssh-agent
jmc       6586  0.0  0.0   6088   900 pts/1    S+   11:29   0:00 grep ssh-agent
jmc@jmc-PC:/etc/apt$ eval "$(ssh-agent)"
Agent pid 6882
jmc@jmc-PC:/etc/apt$ ps aux |grep ssh-agent
jmc       6882  0.0  0.0   5868   472 ?        Ss   11:32   0:00 ssh-agent
jmc       6893  0.0  0.0   6088   896 pts/1    S+   11:32   0:00 grep ssh-agent
###
whene you close the terminal, the ssh-agent is gone

to add key to ssh-agent
	ssh-add ~/thePathToTheKeyThatWeWantToAdd
only the first time he will ask you for the passphrase (unless u don't close the terminal)


to accepts connection (server side)  ssh deamon --> sshd
which sshd
to check the status 
	systemctl status sshd
###
jmc@jmc-PC:/etc/apt$ systemctl status sshd
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2022-03-19 14:33:18 EDT; 21h ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 5012 (sshd)
    Tasks: 1 (limit: 4915)
   Memory: 3.2M
   CGroup: /system.slice/ssh.service
           └─5012 /usr/sbin/sshd -D

Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.
###

to change the port on command line
	ssh -p 2222 root@serverToReach


//////////////
after the clone you have to 
	npm i 
	ng serve

//////////////////////
to install ghpage for angular module
	npm i -g angular-cli-ghpages

	ng build --prod --base-href "https://username.github.io/repoName/"
to upload project over github
	angular-cli-ghpages
another way to do the upload
	ngh --dir=dist/repoName

j'ai ajoute des modifs dans package.json.scripts
pour builder
	npm run build
	npm run deploy

