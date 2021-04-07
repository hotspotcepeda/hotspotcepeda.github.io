---
title: "Programar reinicio en OpenWRT"
date: 2016-03-03
draft: false
description: "Programar reinicio en OpenWRT"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- OpenWRT
- router neutro
- reinicio
categories:
- red
series:
- Red comunitaria
image: images/feature1/openwrt.png
---
{{< notice info >}}
https://wiki.openwrt.org/doc/howto/cron
{{< /notice >}}
Me están dando guerra una serie de router. El usuario se queja de que de vez en cuando el router pierde la salida a internet, una de las veces que estaba teniendo el problema me conecté y era como que perdía la ip WAN del cliente dhcp, la red local seguía funcionando y la wifi también,la parte lan estaba ok y he pensado que programando un reinicio diario de madrugada se podía arreglar el problema. (No será la mejor solución pero se arregló el problema)

Es fácil solo hay que decirle al cron del router que se reinicie a una hora en la que no tenga actividad, por ejemplo las 04:30 de la madrugada. Primero hay que ponerlo en hora, activar el cliente ntp para que el trasto se ponga en hora cada vez que se encienda… si no tiene ntp configurado se va a reiniciar a lo loco, y queremos que se reinicie cada día a las 04:30. 
```
Este es el formato de cron:

*     *     *     *     *  comando a ejecutar
-     -     -     -     -
|     |     |     |     |
|     |     |     |     +----- Día de la semana (0-7)
|     |     |     +----------- Mes (1 - 12)
|     |     +----------------- Día del mes (1 - 31)
|     +----------------------- Hora (0 - 23)
+----------------------------- Minuto (0 - 59)
``` 
Para insertar una nueva tarea en el cron:
```
# crontab -e
```
Se nos abre un fichero en el que ponemos esto, se trata de confirmar la hora antes de hacer el reboot para no dejar un reinicio en bucle y cargarse el trasto.
```
# Reinicio todos los dias a las 04:30
# Nota: Para evitar el bucle de reinicio infinito, espera 70 segundos
# y crea un archivo en /etc comprueba que la fecha de creación 
# marca las 4:31 antes de ejecutar el comando de reinicio de cron.
30 4 * * * sleep 70 && touch /etc/banner && reboot
```
Para salir de la edición del archivo usando vi:
Presionar tecla Esc y escribir :wq , para salir sin guardar :q!
Arrancamos y activamos el servicio:
```
# /etc/init.d/cron start
# /etc/init.d/cron enable
```
Comprobamos que permanece y esperamos que funcione. Hasta mañana!
```
# crontab -l
```
Al día siguiente ejecutamos un:
```
# uptime
```
Para ver cuanto lleva funcionando.
