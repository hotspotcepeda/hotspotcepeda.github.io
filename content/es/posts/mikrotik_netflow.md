---
title: "MikroTik netflow"
date: 2017-05-16
draft: false
description: "Configuracion de netflow en MikroTik servidor Debian nprobe"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- nprobe
- debian
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
## Configuración en MikroTik

![Configuración netflow MikroTik](/images/post/netflow_mikrotik.png)

## Configuración en servidor debian jessie:
```
# apt-get update && apt-get upgrade
wget http://apt-stable.ntop.org/14.04/all/apt-ntop-stable.deb
sudo dpkg -i apt-ntop-stable.deb
sudo apt-get update
sudo apt-get install nprobe
config nProbe
```
```
sudo nano /etc/nprobe/nprobe-none.conf

--zmq="tcp://127.0.0.1:1234"
--collector-port=5556
-n=none
-i=none


sudo touch /etc/nprobe/nprobe-none.start
sudo service nprobe start
```

Instalacion de ntopng
```
wget http://apt-stable.ntop.org/14.04/all/apt-ntop-stable.deb
sudo dpkg -i apt-ntop-stable.deb
apt-get install topng
 
config ntopng

nano /etc/ntopng/ntopng.conf

-G=/var/run/ntopng.pid
-i tcp://127.0.0.1:1234
 
sudo touch /etc/ntopng/ntopng.start
sudo service ntopng start
reboot
```
 
## DOCKER
```
docker run -d --name nprobe --net=host xdrum/nprobe nprobe --collector-port 5556  --zmq tcp://*:1234   -i none
docker run -d --name ntopng --net=host -p 3000:3000 vostro/ntopng ntopng -i tcp://127.0.0.1:1234
```