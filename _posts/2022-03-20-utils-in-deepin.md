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
> **Status** `in progress`
{: .prompt-danger }

Liste des outils que j'utilise souvent dans mon travail. 

I add somereferences temporary handel restapi error[^ref1],  how to add UUID in DB[^ref2] and generator [^ref3].


# TODO after installing Deepin

  - [What to do after installing Deepin OS](https://www.youtube.com/watch?v=b55pe3zXrPEhttps://www.youtube.com/watch?v=b55pe3zXrPE)
  - [10 Things To Do After Installing Deepin 20.1](https://www.youtube.com/watch?v=2h1XNcHmC8g)
  - [TOP 15 Things You MUST DO After Installing Deepin Linux 20.3](https://www.linuxadictos.com/en/despues-instalar-deepin-os.html)

# java
> **Status** `confirmed`
{: .prompt-info }

1. [My first ref](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-debian-10)

# Firefox 
> **Status** `confirmed`
{: .prompt-info }

[Used resource](https://support.mozilla.org/en-US/kb/install-firefox-linux), in section - **Install Firefox from Mozilla builds (For advanced users)**	

[link to download](https://www.mozilla.org/en-CA/firefox/linux/)

> **Info** 
- use `sudo` if you are not a root user
{: .prompt-tip }

Extract the contents of the downloaded file by typing:
```console
$ tar xjf firefox-*.tar.bz2
```
Move the uncompressed Firefox folder to /opt:
```console
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
> **Status** `confirmed`
{: .prompt-info }
From the downloader pkg, using packge installer. [Link to download](https://dbeaver.io/download/)

# STS Tool
> **Status** `not confirmed`
{: .prompt-warning }
from Deepin installer 

> **Entre STS er JAVA**
  pour ne pas laisser les mdp dans application.properties  voir le projet ses
  - dans Run --> Run configuration --> (x)=Arguments
    - add in VN arguments:
      `-Dspring.mail.password=tomMDP`
  {: .prompt-danger }
mvn spring-boot:run -Dspring-boot.run.arguments="toto --spring.mail.password=yzrtgrumdpnsgnab"

to run jar file
```console
$ java -jar nameOfJarFile
```
if error `error - No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?`, take a look to tihis link 
[source to resolve error](https://www.studytonight.com/post/solved-no-compiler-is-provided-in-this-environment-perhaps-you-are-running-on-a-jre-rather-than-a-jdk)

1. Run as --> Maven Test
2. to run jar file with arg 
  - in this case i add the email psw:
```console
java -jar ses-0.0.1-SNAPSHOT.jar --spring.mail.password=yzrtgrumdpnsgnab
```
  - in this case i add the email psw and the activ envirement:
```console
java -jar ses-0.0.1-SNAPSHOT.jar --spring.mail.password=yzrtgrumdpnsgnab spring.profiles.active=prod
```

---
## Reverse Footnote

[^ref1]: java - error handler in springboot restapi: <https://www.youtube.com/channel/UCbA2jT41CAlZF_V1V-iSaCQ>
[^ref2]: [how to use UUID as id](https://www.codementor.io/@petrepopescu/how-to-use-string-uuid-in-hibernate-with-mysql-1jrhjh6ef5)
[^ref3]: [generator](https://theonegenerator.com/)  \ (o-o) /
 

