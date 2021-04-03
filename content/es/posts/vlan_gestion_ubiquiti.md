---
title: "Configurar vlan de gestión en Ubiquiti"
date: 2017-04-15
draft: false
description: "Configurar vlan de gestión en Ubiquiti"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- ubiquiti
- gestión
categories:
- red
series:
- Red comunitaria
image: images/feature1/UBNT_logo.png
---
{{< notice info >}}
http://ca.wiki.guifi.net/wiki/Gestio_de_antenes_ubnt_a_trav%C3%A9s_de_VLAN_de_gesti%C3%B3
{{< /notice >}}

Lo que queremos es mantener gestión de las antenas de manera directa. En los equipos Ubiquiti se permiten hacer vlans, desde la version 5.5.

Se define una VLAN de gestión: vlan666

Se define una VLAN de tráfico general: vlan10

Se configura la antena en bridge, se le añade dos interfaces VLANs, y se configura la IP manual en la LAN0.X X = vlan ID (en nuestro caso 666)

![Configuración de VLAN de gestion en Ubiquiti](/images/post/vlan_gestion_ubnt.png)

Se configura la VLAN de gestión a la RB: añadir VLAN a la interfaz de la antena

Se añade la VLAN de gestión de una o todas las antenas al bridge WLAN / Lan. De esta manera podremos darle direccionamiento 10.x de gestión del supernodo

**Este modelo lleva una ventaja muy interesante, y es que podemos configurar un servidor DHCP para todo el management de las antenas, ya que las integramos al bridge WLAN / LAN**




