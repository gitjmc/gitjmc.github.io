---
title: Common Commands
author:
  name: jmc
  link: https://github.com/cotes2020
date: 2022-03-05 20:24:00 -0500
categories: [Linux, CLI]
tags: [cli, linux]
---

C'est la liste de commandes qui pourront me servir (pour ne pas les oublier) \o/

## Liste des commandes

### pwd

Print working directory

```console
$ pwd
/home/jmc/dev/tempo/jekyll/gitjmc.github.io
```

### cp
Copy files and folders ...

```console
$ cp file_to_copy.txt new_file.txt

$ cp -r dir_to_copy/ new_copy_dir/
```
> Remember that in Linux, folders end with a forward slash `/`.
{: .prompt-info }

### rm
Remove files and folders ...

```console
$ rm file_to_ermove.txt

$ cp -r dir_to_remove/

$ rm -rf dir_with_content_to_remove/
```
### grep
```console
$ history | grep 'exec'

```
### scp
Securely copy files and directories between two locations, [How to Use SCP Command to Securely Transfer Files](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/).
Another interesting link, [optimiser les recherches](https://www.linuxtricks.fr/news/10-logiciels-libres/352-ls-grep-wc-l-find-optimiser-les-recherches/).
```console
$ history | grep 'exec'

```
### vim
How can i save a file opened in vim as the wrong user
>  `E45` readonly option is set (add ! to override).
{: .prompt-danger }

```console
:w !sudo tee %
```

Explanation:
  - :w – write
  - !sudo – call shell sudo command
  - tee – the output of write (:w) command is redirected using tee
  - % – current file name

another way
```console
press Esc, then type :qa! and press Enter
```
### uninstall
for example to uninstall libreoffice
```console
$ sudo apt-get purge libreoffice*
```
