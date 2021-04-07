---
title: "Ejecutar aplicación por ssh"
date: 2017-04-30
draft: false
description: "Como ejecutar aplicaciones por ssh de manera remota"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- debian
- linux
- ssh
- gestion
categories:
- Linux
series:
- Red comunitaria
image: images/feature1/debian_logo.png
---
## SERVIDOR

Agregar en
```
# nano /etc/ssh/sshd_config

X11Forwarding yes

# service ssh restart
```
## CLIENTE
```
$ ssh -X -Y user@maquina -p puerto aplicación
```