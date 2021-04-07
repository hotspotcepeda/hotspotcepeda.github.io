---
title: "NanoStation M5 gateway internet"
date: 2017-01-21
draft: false
description: "Configuración qMp para antena gateway con salida a internet"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- OpenWRT
- qMp
- ubiquiti
- wifi
- cepeda la mora
categories:
- red
series:
- Red comunitaria
image: images/feature1/qmp_logo.png
---
Pasos para usar una antena Ubiquiti Nano Station en una red qmp con Lan y Wan. Las configuraciones de Vlan las vamos a hacer desde el menú de administración en el apartado de switch e interfaces.

La Lan es servidor de dhcp, con la configuración inicial tiene la ip 172.30.22.1/24 user/password root/13f y nos proporciona una IP de ese rango al conectarnos. Tenemos conectado en el POE que va a la boca Main del router y viene como la Vlan1

La Wan es cliente de dhcp, conectamos la boca Secondary a un servidor de dhcp como nuestro router domestico de ASDL o Fibra.  Vamos a crear una nueva Vlan2 y asociarla a la Wan.

Partimos de la configuracion inicial que tenemos nada mas instalar el firmware de qmp.cat

qMp-Guifi_3.2-Clearance_Ubiquiti_NanoStation-M5-XW_sysupgrade_20151026-1823

![Configuración default de VLAN SWITCH](/images/post/configuracion_default_qmp.png)

Modificación en switch.

Para que funcione la Wan no vale con agregar una nueva Vlan a eth0.2 al port 1,

Solo me funciona con normalidad cuando se elimina la Vlan 12 y se agrega una nueva Vlan 2 , revisar la parte de interfaces y firewall. Untagged.

Modificación de interfaces.

![Configuración para WAN de VLAN SWITCH](/images/post/configuracion_vlan_wan_qmp.png)

Asegurarse que la interface se está conectando a internet por la interface física Secondary eth0.2 = Vlan 2 