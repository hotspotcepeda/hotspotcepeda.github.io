---
title: "NanoStation M5 qMp"
date: 2016-05-03
draft: false
description: "Instalar qMp en NanoStation M5"
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
- internet
- cepeda la mora
- NanoStation M5
categories:
- red
series:
- Red comunitaria
image: images/feature1/qmp_logo.png
---
{{< notice info >}}
**¿Que qué es qMp?** Visite http://qmp.cat/Inicio para conocerlo. 
Esta es la página de desarrollo proyecto https://dev.qmp.cat/projects/qmp  **🙌**
{{< /notice >}}
{{< notice warning >}}
27/03/2021 **Ojo con las nuevas remesas de de equipamiento Ubiquiti**, las nuevas revisiones 2000K y superiores de NanoStation Loco M5 no funcionan ni con OpenWRT ar71xx ni con OpenWRT ath79, ni tan siquiera probando OpenWRT ath79 para Bullet M XW que sí funciona en las Rocket M XW. Al parecer hay dos arquitecturas distintas que sirven para distintas revisiones de hardware de Ubiquiti .La ar71xx es la de siempre, antes funcionaba, la ath79 es la nueva. No sé sabe todavía la manera de flashearlas. 
{{< /notice >}}
![NanoStation M5](/gallery/red/NSM5-p.png)
## Preparación
Si el trasto viene con un firmware XW.v5.5.10-u2.28005.150723.1358  hacer ftp de qmp directamente.

Si viene con un firmware superior hacer downgrade a la versión XW.v5.5.10-u2.28005.150723.1358 desde el menú de la web.

A tener en cuenta:
* Conectar más de 1 router como gw a la red qMp no funciona bien, si se quiere conectar a otra wan vale, pero no a la misma, hace bucle y asigna pipa 169.
* Despues de flasear y configurar el router para añadirlo a la red qMp es recomendable apagar y encender.
* Kalimotxo + Kalimotxo ok, pero no funciona bien  kalimotxo + RC2 o RC3
```
# apt-get install tftp-hpa # para poder hacer ftp
```
## Cargar el firmware:
- Con el trasto apagado pisar el reset y encender = TRENECITO --> PARPADEO X3 LUCES Y SOLTAR RESET entramos en modo bootloader ftp.
- Fijar una IP en el rango del trasto que es 192.168.1.20/24             
- http://fw.qmp.cat/stable/ para seleccionar el FW ir a directorio que corresponda.
- Selecionar el .bin guifi master factory del cacharro M2 o M5 (presta gráficas de ping a mayores de consumo), 
descargarlo como qMp.bin
```
>tftp 192.168.1.20 desde el directorio donde esta el qMp.bin
>mode binary
>trace on
>put qMp.bin
```
{{< notice warning >}}
Si se interrumpe la alimentación cuando estamos editando el firmware despídete del aparato.
{{< /notice >}}

Se instala el nuevo FW, pasado un tiempo el cacharro se reinicia con el nuevo FW
Poner la interface en auto DHCP, entrarle al cacharro desde el navegador 172.30.22.1  root – 13f

**Los dispositivos se conectan a la red sin hacer nada más quie configurar los parámetros de la wifi, país, canal, hostname e IP.**
Ir a configuración del nodo qMp --> qMp easy setup Cambiamos el modo de roadming a **comunity**
Poner el nombre del nodo de acuerdo al **nombre** del nodo guifi.net o hacer el 1 solo click dede el menu guifi.net de la web qmp.
**IP address** de acuerdo a la de guifi.net con su mascara.

Atento a la denominación del nodo = hostname es qMpX-YYYY
## Atención
Si queremos separar la red en dos o mas:
- Cada red usará un canal, en un primer momento todos están en el mismo canal y misma red.
- Ir a configuración del nodo wireless setings para cambiar **canal** a los equipos que queramos que estén en otra red.
- Hay dos menus de administración del qMp y del WRT / Para descubrir redes qMp hacemos un SCAN desde las redes qMp, son ad-Hoc.
