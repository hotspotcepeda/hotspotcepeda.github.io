---
title: "Guardar log de MikroTik y Ubiqiti en servidor"
date: 2016-11-01
draft: false
description: "Guardar log de MikroTik y Ubiqiti en servidor remoto Linux Debian"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- debian
- linux
- log
- Ubiquiti
categories:
- red
series:
- Red comunitaria
image: images/feature1/debian_logo.png
---
{{< notice info >}}
https://aacable.wordpress.com/2011/11/29/howto-save-mikrotik-logs-to-remote-syslog-server/
Conviene tener los trastos en hora, configure NTP en todas sus máquinas.
{{< /notice >}}
## Cliente Mikrotik
Con ip 10.38.15.10
```
/system logging action
set 0 memory-lines=100
set 1 disk-file-count=30 disk-file-name=MT-log-zaib disk-lines-per-file=500
set 3 remote=10.38.15.3
# 10.38.15.3 ip del servidor syslog-ng :514 default port

/system logging
add action=remote topics=critical
add action=remote topics=error
add action=remote topics=info
add action=remote topics=warning
# registro remoto de lo que se quiera
```
## Cliente Ubiquiti
Con ip 10.38.15.1
![Configuración log remoto Ubiquiti](/images/post/syslog_UBNT.png)

## Servidor 
IP 10.38.15.3 servidor donde se almacenan los LOG
```
#apt-get install syslog-ng
#mkdir /var/log/mikrotik
#mkdir /var/log/ubnt
```
en nano /etc/syslog-ng/syslog-ng.conf agregar y después reiniciar con "service syslog-ng restart"
```
Accept connection on UDP
source s_net { udp (); };
# MIKROTIK ########### esto genera de manera automática un fichero tipo "mikrotik.2016.11.01.log"
# Add Filter to add our mikrotik
filter f_mikrotik { host( "10.38.15.10" ); };
# Add destination file where logs will be stored
#destination df_mikrotik { file("/var/log/mikrotik.log"); };
log { source ( s_net ); filter( f_mikrotik ); destination ( df_mikrotik ); };
destination df_mikrotik {
 file("/var/log/mikrotik/mikrotik.${YEAR}.${MONTH}.${DAY}.log"
# template("${HOUR}:${MIN}:${SEC} ${HOST} ${MSG} ${MSG}\n")
 template-escape(no));
};
# UBNT ########### esto genera de manera automática un fichero tipo "ubnt.2016.11.01.log"
# Add Filter to add our ubnt
filter f_ubnt { host( "10.38.15.1" ); };
# Add destination file where logs will be stored
#destination df_ubnt { file("/var/log/ubnt.log"); };
log { source ( s_net ); filter( f_ubnt ); destination ( df_ubnt ); };
destination df_ubnt {
 file("/var/log/ubnt/ubnt.${YEAR}.${MONTH}.${DAY}.log"
# template("${HOUR}:${MIN}:${SEC} ${HOST} ${MSG} ${MSG}\n")
 template-escape(no));
};
```
## Ver log live
```
tail -f /var/log/mikrotik/mikrotik*.log
```