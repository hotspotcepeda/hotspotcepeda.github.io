---
title: "Compartir salida a internet con guifi.net"
date: 2016-01-06
draft: false
description: "guifi.net y redes comunitarias en comunidades de vecinos"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
categories:
- red
series:
- Red comunitaria
tags: 
- comunidad de vecinos
- guifi.net
- openwrt
image: images/feature1/guifi_net.png
---
Se trata de interconectar a tres vecinos, el escenario es una comunidad de vecinos de pisos donde se van compartir los gastos en infraestructura común de telecomunicaciones y conexión a internet. Como los tres vecinos están en el mismo edificio, se conectan con cable de red, se instala un router central junto al router de proveedor 300/30 Mb y un router de cliente para cada  nodo se está usando firmware openWRT en los equipos.

Router central 10/100/1000 TP-Link TL-WR1043ND

Router cliente 10/100 Nexx WT3020

Router central de segunda mano 20 €, y router de cliente 15x3=45 €. Cable y conectores 20 €

Total unos 85 € en materiales.

La configuración se llevó una hora. El cableado y la instalación menos de 3 horas.

![Esquema comunidad](/images/post/esquema_comunidad1.png)

Las viviendas están conectadas entre si con una red plana y cada una tiene una red wifi privada con salida a internet. La sensación de los vecinos es la de tener un router de proveedor propio pero están compartiendo solo una conexión, pasan de tener 3 conexiones contratadas a pagar una conexión entre 3.

La red está aislada, no está conectada con el resto de usuarios de la red guifi.net. Para conectar con la red guifi.net y compartir servicios comunitarios habría que hacerlo mediante tunel VPN o mediante enlace físico a un segmento de red que ya esté integrado.

La red se está comportando bien, pero no se le está sacando todo el rendimiento, los vecinos nunca alcanzan los 300 Mbit/s. Como los router finales son 10/100 Mbit/s solo llegan unos 80 Mbit/s a cada casa.

La red está limitada en cuanto a su crecimiento como red comunitaria de infraestructura común, para progresar es necesario regular el servicio de salida a internet y abrir las puertas a todas las personas que quieran formar parte de ella. Vamos con un ejemplo: 
Sí el día de mañana otro vecino quiere sumarse a la red comunitaria, con tirar un cable hasta el router central tendría conexión con el resto de vecinos y de manera automática salida a internet. ¿Pero que pasa el día que un vecino toma la decisión de no pagar la parte de la cuota de salida a internet que le corresponde? La solución es fácil, el vecino que tiene el router central desconecta el cable, por que no le parece bien que el resto de los vecinos sigan pagando y este no. 
Mal hecho, ya que cuando se desconecta al vecino moroso se le interrumpe la salida a internet pero además se le deja fuera de la red comunitaria, deja de alcanzar directamente a sus otros vecinos con los que compartía servicios, de p2p, impresoras en red o telefonía IP.
## Solución:
En guifi.net el servicio de salida a internet está regulado de 3 maneras, Hotspot de acceso abierto, PROXY y tunnel VPN.

Los hotspot son puntos de conexión directa por wifi, generalmente estos accesos están limitados en tiempo y caudal se prestan para facilitar conexiones puntuales a internet. Estas redes no tienen cifrado, los SSID de se llaman "guifi.net_AccesoAbierto".

Los proxys son salidas a internet que facilitan los miembros de guifi.net, ceden una porción de ancho de banda. En guifi.net son proxys federados, no están abiertos a todo el mundo, para poder acceder al servicio el propietario o el administrador del proxy tiene que facilitar una contraseña y se mira a quien se da acceso, normalmente a nodos que llevan operativos un tiempo. El caudal es limitado y solo se puede navegar desde dispositivos que admitan configuración de proxy cliente con usuario y contraseña.

Los tunel VPN son enlaces punto a punto desde el cliente hasta la salida a internet, caminos virtuales que trabajan por debajo de los enlaces físicos. Lo ideal seria que todas las zonas tuvieran los tres servicios. Son generalmente un servicio de pago, por que salir a internet cuesta dinero. Los proveedores de servicio de guifi.net tienen servidores de tunel y venden caudales de salida a los usuarios de las redes comunitarias que lo necesitan. De esta forma al usuario que no paga el servicio se le desconecta del tunel de salida a internet, pero sigue teniendo presencia en la red comunitaria guifi.net.


Equipamiento 350 €

Router 10/100/1000, como servidor se VPN para gestionar la salida a internet. RB850Gx2.  + Carcasa CA150

Router 10/100/1000 + Wifi, como router de cliente final. RB951G-2HnD

Al adaptar la red a Giga Bit se asegura un aprovechamiento óptimo de la salida a internet.Este equipamiento permite hacer túneles cliente servidor para salir a internet e interconexión por VPN con otras redes comunitarias como guifi.net. El router MikroTik sustituye al de proveedor eliminando el doble NAT.
## guifi.net
Este es el esquema de red y servicios básicos de un super nodo integrado en guifi.net. Se agrega un servidor y un switch que puede permitir mas conexiones al supernodo, Hotspot y enlaces punto a punto con otras redes o con otros nodos.

![Esquema comunidad guifi.net](/images/post/esquema_comunidad2.png)



