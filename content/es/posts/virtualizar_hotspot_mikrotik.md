---
title: "Virtualizar hotspot MikroTik"
date: 2016-03-15
draft: false
description: "Virtualizando hotspot MikroTik RouterOS con VirtualBox"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- hotspot
- VirtualBox
- RouterOS
- guifi.net
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
## VirtualBox
Vamos a virtualizar RouterOS en una vieja maquina para convertirlo en hotspot MikroTik.
Necesitamos una maquina que tenga más de una interfaz de red, un router con una sola interfaz no tiene mucho sentido.
Un PC Pentium 4 completo se puede encontrar de segunda mano por 50 euros, y tarjetas de red 10/100 las regalan.
En el servidor nos aseguramos que en la bios esté marcada la casilla de virtualización, (la máquina tiene que soportar virtualización  VT-x o AMD-V) ya de paso en la sección de power management que el servidor se encienda si hay una interrupción de la alimentación eléctrica.
También vamos a decirle al servidor que arranque automáticamente nuestro router virtualizado cuando se encienda.
Para ver si nuestra maquina Linux cumple con los requisitos para virtualizar podemos ejecutar unos comandos desde la consola:
```
egrep -c '(vmx|svm)' /proc/cpuinfo  # Nos dice el número de procesadores que soportan virtualización
 4
uname -m  #  Nos dice si el kernel es de 32 o de 64 bit:
 x86_64
```
Primero se instala una Debian y sobre ésta el VirtualBox.
Ahora importamos la imagen [MikroTik-6.29.1.ova](https://www.mediafire.com/file/fp7i4q462o7n595/MikroTik-6.29.1.ova/file) desde el VirtualBox.
Se ajustan las opciones de red de la maquina virtual, en este caso se elige la opción sencilla y se pone la interfaz de red en bridge con la maquina virtual para tener salida a internet.

## MikroTik
Y vamos con la configuración del MikroTik.
Para configurar el router descargar WinBox, esto es gracioso, por que es una aplicación para windows que se usa para gestionar un sistema basado en linux, y de momento no hay WinBox para linux así que tenemos que instalar Wine para poder trabajar desde linux con WinBox. 

## Hotspot
Aquí una guía para HotSpot guifi.net https://guifi.net/node/31382

En nuestro HotSpot vamos a tener 2 modelos de usuarios en “user profiles”.

- **Trial**, es un usuario ocasional con caudal limitado y tiempo de sesión de 1 hora al día, el contador de tiempo se resetea cada 12 horas. El usuario accede a internet con un click.
- **Registrado**, es un usuario que ha solicitado acceso mediante registro, (enviando un mail al administrador) cuenta con un usuario (mail) y una contraseña que le permite el acceso ilimitado en tiempo de sesión. También tiene el caudal limitado.

Firewall, además de las reglas básicas vamos a eliminar el uso de P2P para todos los clientes del HotSpot. 
https://wiki.mikrotik.com/wiki/Basic_universal_firewall_script
Los router MikroTik pueden ejecutar script y tienen un programador de tareas. Esto viene muy bien para hacer copias de seguridad periódicas, enviar log de actividad por mail, programar reinicios automaticos, hacer limpiezas de caché, avisarnos de alarmas o alertas de funcionamiento. 
https://wiki.mikrotik.com/wiki/Scripts
https://wiki.mikrotik.com/wiki/Manual:System/Scheduler

Las restricciones en navegación se van ha hacer en base a la salida a internet que tengamos. El HotSpot de acceso abierto es un servicio limitado o muy limitado, dependiendo del ancho de banda con el que se cuente, está orientado a dar a conocer nuestro proyecto y fomentar el uso de las redes comunitarias como guifi.net.