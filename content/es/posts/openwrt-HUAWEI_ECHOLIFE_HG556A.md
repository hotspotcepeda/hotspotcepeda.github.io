---
title: "OPENWRT EN HUAWEI ECHOLIFE HG556A"
date: 2016-04-08
draft: false
description: "OPENWRT en HUAWEI ECHOLIFE HG556A Una joya!"
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
De como reciclar un viejo router ADSL y convertirlo en un router neutro con software libre. Se pueden usar como firewall, como AP o por entretenimiento para aprender redes porque ofrecen muchas funciones. 
Cambiando el firmware del fabricante a OpenWRT se transforman en router Linux. Dejo un enlace de la lista de hardware compatible https://openwrt.org/toh/start porque seguro que tienes algún router viejo por casa y es compatible. Dependiendo del router es más o menos fácil cargar el firmware, este router que tiene USB es fácil y va genial.

![HUAWEI ECHOLIFE HG556A](/gallery/red/hg556a.png)
https://wiki.openwrt.org/toh/huawei/hg556a

<!--more-->
## 1. Seleccionar firmware:
Seleccionamos el firmware en base al modelo y sacamos el modelo en base al numero de serie del trasto:
Router version	Board model	Wifi chip	Vdf Version *	

Serial number (first 5 digits)	Flash chip erasesize **	cal_data offset	Supported	Notes
A	HG55VDFA VER.C	Atheros AR9223	HG556AVDFA	30462, 30562, 30605, 30608, 30634	0x10000	0xf7e000	Yes	ADSL not supported
VoIP not supported
B	HG556BVDFA	30692, 30693, 31110, 31300	0x20000	0xefe000
C	HG56BZRB VER.A	Ralink RT3062F	HG556CVDFA	30555, 30694, 30695, 31301, 31507, 31525, 31901, 31902, 31935, 32505	0x20000	0xeffe00	Yes
## 2. Instalar el firmware
{{< notice warning >}}
Si en algún momento del proceso al acceder por web al router para cambiar el firmware nos aparece la pagina rosa con el texto:
404 Not Found
File not found.
micro_httpd
Vamos a las preferencias de nuestro navegador, privacidad y limpiamos la cache, esto pasa por que le hacemos un lio al navegador y ya no sabe que página entregarnos, la cache no coincide con la realidad.
{{< /notice >}}
Selecionamos el .bin, como mi numero de serie de mi trasto empieza por 30692 me corresponde la versión B:

openwrt-15.05-brcm63xx-generic-HG556a_B-squashfs-cfe.bin

Pisamos el boton restart del trasto y le damos corriente, mantenemos pulsado unos 15 segundos para acceder a la web de carga del firmware del bootloader.

Nos conectamos en la LAN2 del router y nos ponemos una ip del rango 192.168.1.x/24
Y a cargar el Firmware… Seleccionamos el .bin y Update Software, dice que tarda unos 2 minutos, vemos que el led del power parpadea y se van encendiendo otros led, vamos a dejarlo un buen rato. Esperamos a que la luz del power se quede fija y nos entregue ip.
{{< notice warning >}}
Si se interrumpe la alimentación cuando estamos editando el firmware despídete del aparato.
{{< /notice >}}
## 3. Configuración como router neutro.
Cuando se pone el nuevo firmware solo se puede acceder por telnet y sin contraseña, el SSH se activa cuando se fija el password de root.
Desde PC pedir ip por dhcp:
``` bash
dhclient eth0
```
Nos conectamos a la LAN, vamos al navegador, 192.168.1.1 y ya nos aparece el user password de administración del LUCI que nos sugiere fijar un password de root.

Prestar atención al la IP de la LAN, para que al poner router conectado al router con internet no estén los dos con los mismos rangos..
click en Go to password configuration… para cambiar el password de root, con esto se activa el acceso SSH por consola, si no, solo funciona telnet.

Vamos a convertirlo en router neutro, una boca wan y el resto lan.

Click en network Switch add y lo dejamos así.
![HUAWEI ECHOLIFE HG556A router neutro](/gallery/red/hg556a_neutro.png)
Vamos a Network interfaces y lo dejamos así:
![HUAWEI ECHOLIFE HG556A router neutro](/gallery/red/hg556a_interfaces.png)
Con el servidor de DHCP de la LAN en un rango distinto a la WAN , interface VLAN2  eth0.2
La WAN cliente de DHCP conectada a aun router que entregue IP y con salida a internet, interface VLAN1 eth0.1

En este caso el firmware viene con los drivers WIFI y USB, si hubiera que instalarlos entrar por SSH y ejecutar:
opkg update
opkg install kmod-ath9k
opkg install kmod-usb-ohci kmod-usb2
opkg install usbutils
reboot && exit
lsusb

## 4 Comprobaciones
Se accede a lUCI perfectamente, la wifi es muy buena. 
SoC Broadcom 6358 / 64MB RAM / 16MB Flash.


