---
title: "Ralph en Docker"
date: 2015-04-16
draft: false
description: "Ralph Docker install"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags: 
- ralph
- docker
image: images/feature1/ralph.png
---
https://ralph-ng.readthedocs.io/
Ralph es un sistema de gestión de activos, DCIM y CMDB simple pero potente para centros de datos y back office.

Características:

- realizar un seguimiento de las compras de activos y su ciclo de vida
- sistema de flujo flexible para el ciclo de vida de los activos
- centro de datos y soporte de back office
- Visualización dc incorporada
## Ralph en Docker
``` bash
docker run -i -t --name="mysql_ralph" -v /var/lib/mysql -v /home/ralph/.ralph busybox /bin/sh -c "chown root /home/ralph; chown root /home/ralph/.ralph"
docker run -P -t -i --volumes-from mysql_ralph allegrogroup/ralph:latest /bin/bash /home/ralph/init.sh
docker run -P -p 8000:8000 -t -i --name ralph --mac-address=02:42:ac:11:ff:ff --volumes-from mysql_ralph allegrogroup/ralph:latest
```
Compose
https://ralph-ng.readthedocs.io/en/stable/
Install
Install docker and docker-compose first.
Create compose configuration
Copy docker-compose.yml.tmpl outside ralph sources to docker-compose.yml and tweak it.
Build
Then build ralph:
``` bash
docker-compose build
```
To initialize database run:
``` bash
docker-compose run --rm web /root/init.sh
```
Notice that this command should be executed only once, at the very beginning.
If you need to populate Ralph with some demonstration data run:
``` bash
docker-compose run --rm web ralph demodata
```
Run ralph at the end:
``` bash
docker-compose up -d
```
Ralph should be accessible at http://127.0.0.1 (or if you are using boot2docker at $(boot2docker ip)). Documentation is available at http://127.0.0.1/docs.
If you are upgrading ralph image (source code) run:
``` bash
docker-compose run --rm web /root/upgrade.sh
```
 