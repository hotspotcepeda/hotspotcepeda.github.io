---
title: "OpenWRT COMTREND CT-5365"
date: 2015-10-25
draft: false
description: "OpenWRT en Comtrend CT-5365"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- OpenWRT
- Router neutro
- wifi
categories:
- red
series:
- Red comunitaria
image: images/feature1/openwrt.png
---
De como reciclar un viejo router ADSL y convertirlo en un router neutro con software libre. Se pueden usar como firewall, como AP o por entretenimiento para aprender redes porque ofrecen muchas funciones. 
Cambiando el firmware del fabricante a OpenWRT se transforman en router Linux. Dejo un enlace de la lista de hardware compatible https://openwrt.org/toh/start porque seguro que tienes algún router viejo por casa y es compatible. Dependiendo del router es más o menos fácil cargar el firmware, este router no es fácil y va corto de memoria pero se puede.

![Comtrend_CT-5365](/gallery/red/CT-5365.png)
https://openwrt.org/toh/comtrend/ct5365

<!--more-->
## 1. Seleccionar firmware:
Este es un Router muy parecido al CT-5361 es un poco más moderno, pero no deja de ser una patatilla, ojo puede dar mucho juego aunque tenga poca memoria.
Estos son los ensayos que hice:
##### Attitude Adjustment:
http://downloads.openwrt.org/attitude_adjustment/12.09/brcm63xx/generic/openwrt-96348GW-generic-squashfs-cfe.bin 
Se reinicia el router al navegar por luci 👎
http://downloads.openwrt.org/attitude_adjustment/12.09/brcm63xx/generic/openwrt-96348A-122-generic-squashfs-cfe.bin 
Se reinicia el router al navegar por luci. Por consola responde, se satura cuando navegas por el luci con pinchar un par de pestañas se bloquea a continuación página de tiempo agotado y termina por reiniciarse. 👎
#### Backfire
http://downloads.openwrt.org/backfire/10.03.1/brcm63xx/openwrt-96348GW-generic-squashfs-cfe.bin 
No se enciende el power después de cargar el firmware, no da ip 👎
http://downloads.openwrt.org/backfire/10.03/brcm63xx/openwrt-96348GW-generic-squashfs-cfe.bin 
No se enciende el power después de cargar el firmware, no da ip 👎
http://downloads.openwrt.org/backfire/10.03/brcm63xx/openwrt-96348GW-11-squashfs-bc300-cfe.bin 
No se enciende el power después de cargar el firmware, no da ip 👎
https://dl.dropbox.com/u/4708147/openwrt/bcm63xx/CT-5365/openwrt-96348GW-generic-squashfs-cfe_backfire_10.03.1.bin 
Enlace roto, no pude probar. ✋
#### Barrier Breaker
http://downloads.openwrt.org/barrier_breaker/14.07-rc2/brcm63xx/generic/openwrt-96348A-122-generic-squashfs-cfe.bin 
Se reinicia el router al navegar por luci. 👎
### Backfire (10.03.1, unknown)
Compartido por plumanegra.
http://www.4shared.com/file/yGiVhj8ice/openwrt-96348GW-generic-squash.html 
**Funciona el luci y se mantiene !!!** 👍
## 2. Instalar el firmware
Antes de instalar el firmware conviene visitar la wiki openWRT del modelo http://wiki.openwrt.org/toh/comtrend/ct5365 para echar un vistazo.
Encender el router con el botón de reset pulsado unos 10 segundos para entrar al modo bootloader.
Fijar una IP del rango del router 
``` bash
ifconfig eth0 192.168.1.222
```
Nos conectamos al a la boca X3 por ejemplo y cargamos el .bin, desde el navegador a la ip del portal bootloader 192.168.1.1 
{{< notice warning >}}
Si en algún momento del proceso al acceder por web al router para cambiar el firmware nos aparece la pagina rosa con el texto:
404 Not Found
File not found.
micro_httpd
Vamos a las preferencias de nuestro navegador, privacidad y limpiamos la cache, esto pasa por que le hacemos un lio al navegador y ya no sabe que página entregarnos, la cache no coincide con la realidad.
{{< /notice >}}
Ahora podemos acceder a la web para subir el firmware. Pasados unos minutos se solicita ip por dhcp.
``` bash
telnet 192.168.1.1
passwd root # para fijar el password y activar el SSH
```

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
Para ponerlo como router neutro editamos el /etc/config/network y lo dejamos tal que así, (prestando atención a las MAC, sumar 1 para la MAC de la WAN)
vi /etc/config/network
``` bash
config ‘interface’ ‘loopback’
option ‘ifname’ ‘lo’
option ‘proto’ ‘static’
option ‘ipaddr’ ‘127.0.0.1’
option ‘netmask’ ‘255.0.0.0’

config ‘switch’ ‘eth1’
option ‘enable’ ‘1’
option ‘enable_vlan’ ‘1’
option ‘reset’ ‘1’

config ‘interface’ ‘lan’
option ‘ifname’ ‘eth1.1’
option ‘force_link’ ‘1’
option ‘type’ ‘bridge’
option ‘proto’ ‘static’
option ‘ipaddr’ ‘192.168.1.1’
option ‘netmask’ ‘255.255.255.0’
option ‘nat’ ‘1’
option ip6assign ’60’
option macaddr ’00:1D:XX:XX:XX:31′

config ‘interface’ ‘wan’
option ‘ifname’ ‘eth1.0’
option ‘force_link’ ‘1’
option ‘macaddr’ ’00:1D:XX:XX:XX:32′
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
## 4 Comprobaciones
Guardamos los cambios y reiniciamos la configuración de red con un /etc/init.d/network reload , lo dejamos unos minutos y conectamos la WAN, tener en cuenta que ahora los puertos del router se numeran al revés, en la posición 4X está el port 0, es aquí donde conectamos el internet WAN. 3X 2X y 1X son las patas de LAN.
![Nueva routulación de bocas](/gallery/red/bocas.PNG)
Apagamos y encendemos el router (al reiniciar interfaces se puede bloquear), y a navegar!. 
Ahora podemos configurar cómodamente la WiFi y el resto de opciones desde los menús del lUCI.



