---
title: "Gestión OpenWRT"
date: 2017-05-20
draft: false
description: "Configuraciones y gestión de tráfico en OpenWRT"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- OpenWRT
categories:
- red
series:
- Red comunitaria
image: images/feature1/openwrt.png
---
## Gráficas luci-app-vnstat
A parte de las gráficas en tiempo real que vienen de serie "Realtime Traffic" he puesto el "VnStat Graphs" que almacena en el router gráficas de trafico en las interfaces.
## Mas gráficas, CPU, memoria, temperatura... luci-app-statistics
https://echt.guth.so/persistent-traffic-log-and-more-in-openwrt/
## Smart Queue Management 
luci-app-sqm
Se configura por luci , sirve para limitar el trafico de las interfaces, se instala con varios script de gestión de trafico, en este caso se usa para limitar el caudal de salida a internet se puede usar por ejemplo, para reservar un caudal máximo en la wifi y otro en las interfaces físicas. O para asignar un caudal fijo a las conexiones de la LAN.
## Dinamic DNS luci-app-ddns
Se puede configurar por luci o en el archivo /etc/config/ddns en este caso va a preguntar la ip pública a 
http://checkip.dyndns.com 
y se la dice a changeip.com donde tenemos configurado un nombre de dominio asociado.
```
config ddns 'global'
 option date_format '%F %R'
 option log_lines '250'
 option allow_local_ip '0'

config service 'myddns_ipv4'
 option interface 'wan'
 option enabled '1'
 option service_name 'changeip.com'
 option domain 'dominio.changeip'
 option username 'user_changeip'
 option password 'password_changeip'
 option use_syslog '2'
 option use_logfile '1'
 option ip_source 'web'
 option ip_url 'http://checkip.dyndns.com'
 option force_interval '72'
 option force_unit 'minutes'

config service 'myddns_ipv6'
 option update_url 'http://[USERNAME]:[PASSWORD]@your.provider.net/nic/update?hostname=[DOMAIN]&myip=[IP]'
 option domain 'yourhost.example.com'
 option username 'your_username'
 option password 'your_password'
 option use_ipv6 '1'
 option interface 'wan6'
 option ip_source 'network'
 option ip_network 'wan6'
```
## Cliente mail ssmtp
Para enviar log y alarmas de reinicios. Se configura según el enlace y funciona ok, /etc/ssmtp/ssmtp.conf.
```
root=tu_mail@gmail.com
 hostname=localhost
 mailhub=smtp.gmail.com:587
 FromLineOverride=YES
 UseTLS=YES
 USESTARTTLS=YES
 AuthUser=tu_mail@gmail.com
 AuthPass=tu_password
Y enn /etc/ssmtp/revaliases:

# sSMTP aliases
 #
 # Format:       local_account:outgoing_address:mailhub
 #
 # Example: root:your_login@your.domain:mailhub.your.domain[:port]
 # where [:port] is an optional port number that defaults to 25.
 root:tu_mail@gmail.com:smtp.gmail.com:587
```
## Script reboot.
https://wiki.openwrt.org/doc/techref/initscripts 
Preparamos un script para que se envíe un correo de alarma cuando esté arrancando el router. Seguir los pasos de Automatic email sending.
Script log diario. Envío de ficheros con mpack
```
maillog.txt  
mpack maillog.txt tu_mail@gmail.com
Subject: asunto
```
## CRON
Programar que nos envíe un correo con el log diario. 