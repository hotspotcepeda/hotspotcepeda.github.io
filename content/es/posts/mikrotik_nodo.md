---
title: "Nodo MikroTik vlan"
date: 2017-04-29
draft: false
description: "Configuracion de vlan en nodo MikroTik"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- guifi.net
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
{{< notice info >}}
http://wiki.mikrotik.com/wiki/Manual:CRS_examples
{{< /notice >}}
para una RB850
```
/interface ethernet
set ether3 master-port=ether2
set ether4 master-port=ether2

/interface ethernet switch vlan
add ports=ether2,ether3 switch=switch1 vlan-id=200
add ports=ether2,ether4 switch=switch1 vlan-id=300

/interface ethernet switch port
set ether2 vlan-mode=secure vlan-header=add-if-missing
set ether3 vlan-mode=secure vlan-header=always-strip default-vlan-id=200
set ether4 vlan-mode=secure vlan-header=always-strip default-vlan-id=300
```