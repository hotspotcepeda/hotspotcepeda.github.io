---
title: "OpenWRT COMTREND CT-5361"
date: 2015-01-10
draft: false
description: "OpenWRT en Comtrend CT-5361"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- OpenWRT
- Router neutro
categories:
- red
series:
- Red comunitaria
image: images/feature1/openwrt.png
---
Como reciclar un viejo router ADSL y convertirlo en un router neutro con software libre. Se pueden usar como firewall, como AP o por entretenimiento para aprender redes porque ofrecen muchas funciones. 
Cambiando el firmware del fabricante a OpenWRT se transforman en router Linux. Dejo un enlace de la lista de hardware compatible https://openwrt.org/toh/start porque seguro que tienes algún router viejo por casa y es compatible. Dependiendo del router es más o menos fácil cargar el firmware, este router no es fácil y va corto de memoria pero se puede.

![Comtrend_CT-5361](/gallery/red/CT-5361.png)
https://openwrt.org/toh/comtrend/ct5361

<!--more-->
## 1. Seleccionar firmware:
He intentado instalar un Barrier Breaker 14.07 ya compilado de seguridad wireless pero no he podido hacerlo funcionar, el router se reinicia, lo ponen como que ya tiene lUCI, pero tampoco me ha funcionado el acceso por web.
Lista de firmwares Stable:
Chaos Calmer 15.05	Download images	2015 September	r46767
Barrier Breaker 14.07	Download images	2014 October	r42625
Attitude Adjustment 12.09	Download images	2013 April	r36088
Backfire 10.03.1	Download images	2011 December	r29592
Backfire 10.03	Download images	2010 April	r20728
Kamikaze 8.09.2	Download images	2010 January	r18801
Kamikaze 8.09.1	Download images	2009 June	r16278
Kamikaze 8.09	Download images	2008 September	r14510
Kamikaze 7.09	Download images	2007 September	r7831
Kamikaze 7.07	Download images	2007 July	
Kamikaze 7.06	Download images	2007 June	r7204
White Russian 0.9	Download images	2007 January	r6257

Al final se han seguido los pasos de la wiki openWRT para este modelo y lo tengo funcionando con Backfire (10.03.1, r29592) el lUCI no es tan chulo como el Barrier Breaker pero sirve para trabajar igual de bien.
## 2. Instalar el firmware
Encender el router con el botón de reset pulsado unos 10 segundos para entrar al modo boot loader.
Fijar una IP del rango del router 
``` bash
ifconfig eth0 192.168.1.222
```
Abrir en el navegador la dirección del router 192.168.1.1, seleccionar el fichero .bin cargarlo y esperar pacientemente a que se reinicie.
{{< notice warning >}}
Si se interrumpe la alimentación cuando estamos editando el firmware despídete del aparato.
{{< /notice >}}
## 3. Configuración como router neutro.
Cuando se pone el nuevo firmware solo se puede acceder por telnet y sin contraseña, el SSH se activa cuando se fija el password de root.
Desde PC pedir ip por dhcp:
``` bash
dhclient eth0
telnet 192.168.1.1
```
Fijar password
``` bash
passwd root
```
Editar /etc/config/network.
``` bash
vi /etc/config/network
```
Borramos todo el contenido del archivo y pegamos esto directamente. (Cambiar las MAC por las del router y en la mac de la wan sumar 1).
``` bash
config ‘interface’ ‘loopback’
option ‘ifname’ ‘lo’
option ‘proto’ ‘static’
option ‘ipaddr’ ‘127.0.0.1’
option ‘netmask’ ‘255.0.0.0’

config globals ‘globals’
option ula_prefix ‘xxxx:001c:xxxx::/48’

config ‘switch’ ‘eth1’
option ‘enable’         ‘1’
option ‘enable_vlan’    ‘1’
option ‘reset’          ‘1’

config ‘interface’ ‘lan’
option ‘ifname’ ‘eth1.1’
option ‘force_link’ ‘1’
option ‘type’ ‘bridge’
option ‘proto’ ‘static’
option ‘ipaddr’ ‘192.168.1.1’
option ‘netmask’ ‘255.255.255.0’
option ‘nat’ ‘1’
option ip6assign ’60’
option macaddr ’00:30:xx:xx:xx:7B’

config ‘interface’ ‘wan’
option ‘ifname’ ‘eth1.0’
option ‘force_link’ ‘1’
option ‘macaddr’ ’00:30:xx:xx:xx:7C’
option ‘proto’ ‘dhcp’
option ‘defaultroute’ ‘1’

config ‘switch_vlan’
option ‘device’ ‘eth1’
option ‘vlan’ ‘1’
option ‘ports’ ‘1 2 3 4 5t’

config ‘switch_vlan’
option ‘device’ ‘eth1’
option ‘vlan’ ‘0’
option ‘ports’ ‘0 5t’
```
Guardar y reiniciar la configuración de los interfaces, esto tarda:
``` bash
/etc/init.d/network reload
```
## 4 Comprobaciones
Olvídate de la rotulación de interfaces que traía el router de origen, ahora se quedan así:
![Nueva routulación de bocas](/gallery/red/bocas.PNG)
Conectar internet a la WAN 0 y pc a LAN 1 2 ó 3
``` bash
dhclient eth0
ssh 192.168.1.1
```
Ahora desde el router y desde nuestra maquina se alcanza internet ya se pueden actualizar o instalar paquetes.
``` bash
ping 8.8.8.8
opkg update
```
Para configurar la wifi y resto de opciones ya se puede hacer por lUCI.



