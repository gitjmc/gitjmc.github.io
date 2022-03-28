---
title: Multiple accounts (github and/or bitbucket) on one computer
author:
  name: 
  link: 
date: 2022-03-06 08:46:00 -0500
categories: [cloud, repository]
tags: [dev, tuto, github, bitbucket]
---

How to manage multiple accounts github and bitbucket on the same computer ...\o/...

# Recette

For genratre ssh key
```cosole
ssh-keygen -t ed25519 -C "your@email.com"
```

for more details see this youtube - https://www.youtube.com/watch?v=vSeYsk4WYvg

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

#Nader account
Host jnader
  Hostname github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_jnader

```
{: file='.ssh/config' .nolineno}

```console
$ bundle
```











pour le déploiement
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