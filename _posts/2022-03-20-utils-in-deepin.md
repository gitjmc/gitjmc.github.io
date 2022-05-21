---
title: my Utils in Deepin 20.x
author: 
  name: 
  link: 
date: 2022-05-21 10:25:45 -0400
categories: [Deepin, Utils]
tags: [java, FF, DBeaver]
math: true
mermaid: true
image:
  src: /commons/devices-mockup.png
  width: 400
  height: 250
---
> Statut non débuté.
{: .prompt-danger }

Liste des outils que j'utilise souvent dans mon travail[^ref1].

> Disons que Ça pas mal fonctionné!
{: .prompt-info }


## Solution
---
# java

Look for debian's [**version-releases**](https://en.wikipedia.org/wiki/Deepin) used in deepin.

_buster_ - est la version de Debian de Deepin 20.4

If curl is not installed then you have to do

```console
sudo apt update && sudo apt upgrade
sudo apt install curl
```
1. Uninstall old node version

```console
sudo apt -y remove nodejs
```

2. Add the NodeSource package signing key

```console
KEYRING=/usr/share/keyrings/nodesource.gpg
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | gpg --dearmor | sudo tee "$KEYRING" >/dev/null
# wget can also be used:
# wget --quiet -O - https://deb.nodesource.com/gpgkey/nodesource.gpg.key | gpg --dearmor | sudo tee "$KEYRING" >/dev/null
gpg --no-default-keyring --keyring "$KEYRING" --list-keys
```

3. Add the desired NodeSource repository

```console
# Replace with the branch of Node.js or io.js you want to install: node_6.x, node_8.x, etc...
VERSION=node_16.x
# Replace with the keyring above, if different
KEYRING=/usr/share/keyrings/nodesource.gpg
# The below command will set this correctly, but if lsb_release isn't available, you can set it manually:
DISTRO="$(buster)"
echo "deb [signed-by=$KEYRING] https://deb.nodesource.com/$VERSION $DISTRO main" | sudo tee /etc/apt/sources.list.d/nodesource.list
echo "deb-src [signed-by=$KEYRING] https://deb.nodesource.com/$VERSION $DISTRO main" | sudo tee -a /etc/apt/sources.list.d/nodesource.list
```

4. update package lists and install Node.js

```console
sudo apt-get update
sudo apt-get install nodejs
```

5. chaeck installed version

```console
node --version
npm --version
```

For more details take a look to this [**link**](https://github.com/nodesource/distributions/blob/master/README.md#manual-installation)


### Firefox 

```console
sudo npm install -g @angular/cli
```

```console
ng --version
```

---
## Reverse Footnote

[^ref1]: Voir ce lien: <https://github.com/nodesource/distributions/blob/master/README.md#manual-installation>
[^ref2]: Içi avant ne veut pas dire après.  \ (o-o) /
 

