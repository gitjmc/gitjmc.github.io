---
title: Common CLI on linux 
author:
  name: 
  link: 
date: 2025-03-09 07:35:00 -0400
categories: [DEV]
tags: [cli]
render_with_liquid: false
---

### Create non-root user

connecting to remote host with the same user on local machine
Try `root` 

```bash
adduser jmc
```

```bash
usermod -aG sudo jmc
```

```bash
sudo su jmc
```

```bash
whoami
```

```bash
cd ~
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```
from the local

```bash
cat ~/.ssh/id_rsa.pub
```
copy public key and add it into `authorized_keys`

copy the public key to the remote host in authorized_keys


with root user add this command

```bash
usermod -aG sudo jmc
```


### next command ...



