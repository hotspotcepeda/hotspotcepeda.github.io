---
title: "Cliente SNMP linux"
date: 2016-10-30
draft: false
description: "Configuracion de cliente SNMP en linux Debian"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- debian
- jessie
- SNMP
- linux
categories:
- red
series:
- Red comunitaria
image: images/feature1/debian_logo.png
---
## Instalación
```
# apt-get install snmp snmpd snmp-mibs-downloader
```
## Configuracón
Cambiar en nano /etc/snmp/snmpd.conf:
```
agentAddress udp:127.0.0.1:161
...
rocommunity public default -V systemonly
```
por:
```
agentAddress udp::161
...
rocommunity public

```
cambiar en nano /etc/snmp/snmp.conf:
```
mibs :
```
por:
```
#mibs :
```

**Reiniciar snmpd**
```
# service snmpd restart
```
## Comprobar:
```
snmpwalk -v 2c -c public IP
```