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
> **Status** `draft`
{: .prompt-danger }

Liste des outils que j'utilise souvent dans mon travail[^ref1].


# TODO after installing Deepin

  - [What to do after installing Deepin OS](https://www.youtube.com/watch?v=b55pe3zXrPEhttps://www.youtube.com/watch?v=b55pe3zXrPE)
  - [10 Things To Do After Installing Deepin 20.1](https://www.youtube.com/watch?v=2h1XNcHmC8g)
  - [TOP 15 Things You MUST DO After Installing Deepin Linux 20.3](https://www.linuxadictos.com/en/despues-instalar-deepin-os.html)

# java
> **Status** `not confirmed`
{: .prompt-warning }

installer java (source - https://www.youtube.com/watch?v=pkL-EtMbR3w)
download java .deb
double click ( debian package will do the rest :) )
open .bachrc and add the following line

export PATH="/usr/lib/jvm/jdk-18/bin:$PATH"
in the terminal 
```console
$ . .bashrc
$ java -version
$ javac -version
```
> Not sure about the result (must be checked).
{: .prompt-danger }


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


# Firefox 
> **Status** `not confirmed`
{: .prompt-warning }

FF - source - https://support.mozilla.org/en-US/kb/install-firefox-linux	

section - Install Firefox from Mozilla builds (For advanced users)	

link to download - https://www.mozilla.org/en-CA/firefox/linux/

Extract the contents of the downloaded file by typing:
```console
$ tar xjf firefox-*.tar.bz2
```
Move the uncompressed Firefox folder to /opt:
```console
$ mv firefox /opt
$ sudo mv firefox /opt
```

Create a symlink to the Firefox executable:
```console
$ sudo ln -s /opt/firefox/firefox /usr/local/bin/firefox
```

Download a copy of the desktop file
```console
$ sudo wget https://raw.githubusercontent.com/mozilla/sumo-kb/main/install-firefox-linux/firefox.desktop -P /usr/local/share/applications
```
Some links to get more more secure and privacy ...
  - link1: <https://www.youtube.com/watch?v=NH4DdXC0RFw>
  - link2: <https://www.youtube.com/watch?v=tQhWdsFMc24>
  - link3: <https://restoreprivacy.com/firefox-privacy/>
  - link4: <https://support.mozilla.org/en-US/products/firefox/privacy-and-security>
  - link5: <https://www.vpnmentor.com/blog/the-complete-guide-to-privacy-on-firefox-browser/>
  - link6: <https://www.lifewire.com/firefox-security-tips-and-tools-2487972>


> privacy setup
1. FF default browser
2. Enable DNS over HTTPS
3. clean home page
4. only DuckDuckGo
5. Privacy and security
	5.1 select Custom and disable all of them
	5.2 send wesites ... --> always
	5.3 login and pwd --> i didn't touch until i found a good password manager
	5.4 history --> enable "Clear history when Firefox closes" in "Use custom settings for history"
	5.5 Firefox Data Collection and Use --> disabled every thing
6. install extensions
	6.1 Privacy Badger
	6.2 Https everywhere
	6.3 FF multi-account containers
{: .prompt-info }



# synergy
> **Status** `not confirmed`
{: .prompt-warning }
from the downloader pkg, use packge installer 

# dbever-CE
> **Status** `not confirmed`
{: .prompt-warning }
from the downloader pkg, use packge installer 

# STS Tool
> **Status** `not confirmed`
{: .prompt-warning }
from Deepin installer 

---
## Reverse Footnote

[^ref1]: Voir ce lien: <https://github.com/nodesource/distributions/blob/master/README.md#manual-installation>
[^ref2]: Içi avant ne veut pas dire après.  \ (o-o) /
 

