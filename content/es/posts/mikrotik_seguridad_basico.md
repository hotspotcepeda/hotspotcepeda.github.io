---
title: "Seguridad básica con MikroTik"
date: 2016-06-06
draft: false
description: "Medidas básicas de seguridad con MikroTik al poner en producción"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
{{< notice info >}}
http://gregsowell.com/classes/mtk-security/GregSowell-mikrotik-security.pdf
**Si el router está conectado a internet puedes tener seguro que lo van a intentar dominar de todas las maneras posibles.**
{{< /notice >}}
## System -> Password
Poner una contraseña segura, con PassGen se pueden generar passwords seguras, pero **no olvidar !**
## System -> Users
Es buena idea cambiar el nombre del usuario administrador, Cambiar admin por lo que sea. Los zombis están todo el santo día dando la matraca y probando con usuarios y password default. **no olvidar !**
## Ip -> Services
Aquí es donde se abren y cierran las puertas de gestión de la RB. Es recomendable deshabilitar todo lo que no se use y personalizar lo que se use. 
Es buena idea cambiar el puerto por defecto del winbox :8291 a otro puerto **no olvidar !**. 
Para invocar al equipo desde winbox por otro puerto  IP_DE_LA_RB:PUERTO.. 
También  se pueden permitir gestión solo a ip o redes concretas.  Si  queremos activar el FTP o otro servicio hacerlo solo cuando se valla a usar y por un puerto distinto para que no se identifique el servicio, porque si estás conectado a internet te van a escanear los puertos todo el rato.
Hay configuraciones para detectar y parar los escaneos de puertos, ataques Syn Flod y los ataques ssh.
## Ip -> Neighborgs
Deshabilitar las interfaces no confiables, se genera broadcast para descubrir equipos.
## Tools -> BTest Server
Deshábilitar todo y habilitar solo cuando se use.
## System Logging
EL log que guarda la RB es limitado, lo recomendable es usar otro servidor para ir guardando los log.

https://aacable.wordpress.com/2011/11/29/howto-save-mikrotik-logs-to-remote-syslog-server/

En el router indicar la ip del servidor de logs y los temas que queremos enviar al servidor.

En el servidor:
```
# apt-get update
# apt-get install syslog-ng
# mkdir /var/log/mikrotik
# touch /var/log/mikrotik/mikrotik.log
# nano /etc/syslog-ng/syslog-ng.conf
```
Agregamos esto antes de la sección de sources
````
##
# Aceptar conexiones UDP
source s_net { udp (); };
# agragar filtro de la mikrotik
filter f_mikrotik { host( "IP_DE_MIKROTIK" ); };
# lugar donde se van a ir guardando los log
destination df_mikrotik { file("/var/log/mikrotik/mikrotik.log"); };
log { source ( s_net ); filter( f_mikrotik ); destination ( df_mikrotik ); };
##
```
Guardar archivo y reiniciar servicio
```
# service syslog-ng restart
```
Para ver si nos funciona el invento hacer algo en la RB y ver el archivo.
```
# cat /var/log/mikrotik/mikrotik.log
```
Para ver en live el log de la mikrotik
```
# tail -f /var/log/mikrotik/mikrotik.log
```
Para hacer rotación diaria de log, editar y agregar al principio del archivo.
```
# nano /etc/logrotate.d/syslog-ng
/var/log/mikrotik/mikrotik.log
{
 daily
 rotate 90
 compress
 notifempty
 missingok
 sharedscripts
 /etc/init.d/syslog-ng restart
 endscript
}
```
Guardar archivo y reiniciar servicio
```
# service syslog-ng restart
```