---
title: ssh
author:
  name: 
  link: 
date: 2022-04-17 18:02:00 -0400
categories: [Linux, ssh]
tags: [ssh, sshd]
---

This post will try to guide you how to troubleshooting ssh in Deepin[^ref1].
Take a look to tihis video - OpenSSH Full Guide - Everything you need to get started![^ref3].

## Connect to server via ssh
   Use this command to log: `ssh Username@192.168.2.26`
   where "Username" is the user name on machine `192.168.2.26`. You may need to provide the password for this user during login.

````console
jmc@jmc-PC:~$ ssh 192.168.2.26
ssh: connect to host 192.168.2.26 port 22: Connection refused
````
Here we have an error
> ssh: connect to host 192.168.2.26 `port 22`: Connection refused.
{: .prompt-danger }


## check if SSH Service is down
````console
$ sudo service ssh status
````
You can use the command `sudo service ssh restart` to try to get it runing again.
## check which port is being used for SSH
````console
$ grep Port /etc/ssh/sshd_config
````
The command line should return the correct port.

## check if the Port You’re Trying to Use Is Closed
````console
$ sudo lsof -i -n -P | grep LISTEN
````
This command should return a list of ports with the LISTEN state. Ideally, you want to see port 22 or your server’s custom SSH port listed here. If it’s not, you’ll need to reopen the port in order to connect to your server.

## SSH Isn’t Installed on Your Server

## Firewall Settings Are Preventing an SSH Connection
````console
$ sudo iptables-save # display IPv4 rules
$ sudo ip6tables-save # display IPv6 rules
````
Your results will vary, but you’ll want to look for these elements to determine if your firewall is blocking SSH connections:

 - dport 22: This refers to the destination port, which for SSH is usually port 22 (reminder: Kinsta doesn’t use this port number).
 - REJECT: This would indicate that connections are being refused from the specified destination.
 - DROP: Like REJECT, this means that connections to the relevant port are being blocked.

If you search the results of the commands above for dport 22, you should be able to determine if your firewall is preventing an SSH connection. If so, you’ll have to change the rules to accept requests.


## SSH service
### Installation
SSH consists of SSH clicent (openssh-client) and SSH server (openssh-server)[^ref2].

If you just want to log in to other machine using SSH, only openssh-client is neened. deepin has it installed by default; if not, execute in terminal:
````console
$ sudo apt-get install openssh-client
````
If you would like to provide SSH service on a server, execute in terminal:
````console
$ sudo apt-get install openssh-server
````
### Uninstallation
````console
$ sudo apt-get remove openssh-client
$ sudo apt-get remove openssh-server
````
### Configuration
Make sure that SSH server has been started:
````console
$ ps -e |grep ssh
````
A process named "sshd" in the output indicates that SSH server has been started successfully. If no one presents, execute:

````console
$ sudo /etc/init.d/ssh start 
````
or
````console
$ sudo service ssh start
````
The configuration files of SSH server is /etc/ssh/sshd_config, where you can define the port, indentity files and other parameters. The default port for SSH service is 22, you may want to change it to another value, saying 222.

Then restart SSH service:
````console
$ sudo /etc/init.d/ssh stop
$ sudo /etc/init.d/ssh start
````
Now you can log in to the server using SSH:
````console
$ ssh Username@192.168.1.112
````
where "Username" is the user name on machine "192.168.1.112". You may need to provide the password for this user during login.

## Transfer files between two Unix computers
Copy from local to another unix pc

````console
$ scp test.txt jmc@192.168.2.25:~/dev/java
$ sudo scp test.txt jmc@192.168.2.25:~/dev/java

$ scp -r ~/dbbackup jmc@192.168.2.25:~/
$ sudo scp -r ~/dbbackup jmc@192.168.2.25:~/
````

## Frequently asked question
#### View and manage logged-in user
Use `w` and `pkill` command to see active user in the system.



## Reverse Footnote

[^ref1]: [**link to page**](https://kinsta.com/knowledgebase/ssh-connection-refused/)
[^ref2]: [**link **](https://wiki.deepin.org/index.php?title=SSH_service&language=en)
[^ref3]: [**OpenSSH Full Guide - Everything you need to get started! **](https://www.youtube.com/watch?v=YS5Zh7KExvE)


