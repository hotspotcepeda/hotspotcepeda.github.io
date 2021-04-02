---
title: "Hotspot MikroTik con proxy transparente"
date: 2017-01-17
draft: false
description: "Un hotspot es una buena manera para encontrar alguien con quien compartir salida a internet"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- hotspot
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
{{< notice info >}}
La idea es poner un ap en 2.4 Ghz con un portal cautivo para tratar de dar a conocer el proyecto y ver si se anima alguien en el barrio a compartir la salida a internet.
{{< /notice >}}
Antes de empezar vamos a configurar los servicios básicos.
## DNS
Para sacarle partido a la RouterBoard se configura una cache DNS
```
/ip dns
set allow-remote-requests=yes cache-size=4096KiB servers=91.239.100.100,89.233.43.71
```
Rechaza peticiones dns externas:
```
/ip firewall filter
add chain=input in-interface=ether1 protocol=udp dst-port=53 action=drop
add chain=input in-interface=ether1 protocol=tcp dst-port=53 action=drop
```
Y en el firewall una nueva redirección interna del puerto 53
```
/ip firewall nat
add action=redirect chain=dstnat comment="Cacheo de DNS" dst-port=53 protocol=udp to-ports=53
```
## NTP 
```
/system ntp client
set enabled=yes primary-ntp=130.206.0.1 secondary-ntp=130.206.3.166
```
## USB
Como vamos a montar un proxy que lo que hace básicamente es cachear las conexiones y almacenarlas para entregar inmediatamente, vamos a preparar el espacio de almacenamiento, en este caso un pendrive de 1Gb conectado a una RB951.
Para que la RB se entienda con la memoria USB tenemos que formatearlo desde la propia RB, en Fat32, con ext3 no funcionara. Conectamos nuestro pendrive. System->Disks
## Proxy
Ahora vamos a preparar la parte del proxy en IP->Web Proxy, con el Proxy ya trabajando lo vamos a hacer transparente, pasamos las peticiones del puerto 80 al 8080.
IP->Firewall pestaña de NAT Nueva regla
REDIRECCIÓN DE PETICIONES :80 AL :8080 (puerto del proxy cache TRANSPARENTE), SOLO PARA ether5 (puerto de captación del hotspot)

Proxy cache:
```
 > ip proxy print 
                 enabled: yes
             src-address: ::
                    port: 8080
               anonymous: no
            parent-proxy: ::
       parent-proxy-port: 0
     cache-administrator: webmaster
          max-cache-size: unlimited
   max-cache-object-size: 2048KiB
           cache-on-disk: yes
  max-client-connections: 600
  max-server-connections: 600
          max-fresh-time: 3d
   serialize-connections: no
       always-from-cache: no
          cache-hit-dscp: 4
              cache-path: disk1


/ip proxy
set cache-on-disk=yes cache-path=disk1 enabled=yes
```
Y en el firewall una nueva redirección del puerto 80 al 8080
```
/ip firewall nat
add action=redirect chain=dstnat dst-port=80 protocol=tcp to-ports=8080
```
# Hotspot
Generamos el hotspot IP -> HOTSPOT
Yo antes de hacer el SETUP del hotspot asigno IP a la interface de captación, en este caso ether5, le he asignado la ip 192.168.100.1/24 que será la gateway de los clientes del hotspot. Creo un pool por ejemplo de 192.168.100.100-192.168.100.200 y un servidor DHCP que use el pool que hemos creado.

Después en el wizard del hotspot, solo hay que pulsar en todo siguiente, cuando nos pregunta el nombre del DNS ponemos "guifi.net_Acceso_Abierto" será la dirección que aparezca en la página de redirección, el archivo login.html que encontraremos en FILES.

Cuando termina el wizard del hotspot vemos que se han creado una serie de regla en IP->Firewal Filter Rules, Nat y en más sitios...

Tener en cuenta que si el usuario no pasa por el portal cautivo no va a tener salida a internet.

## Securizar
Es conveniente seguir las indicaciones de seguridad.
https://wiki.mikrotik.com/wiki/Manual:Securing_Your_Router
https://hotspotcepeda.github.io/es/posts/mikrotik_seguridad_basico/