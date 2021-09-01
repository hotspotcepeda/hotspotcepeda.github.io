---
title: "Autoprovisión"
date: 2021-04-21
draft: false
description: "Autoprovisión de nodo qMp"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- guifi.net
- qMp
- mesh
- cepeda la mora
- wifi
- autoprovision
categories:
- Ayto
series:
- Red comunitaria
image: images/feature1/flowchart.png
---
Por si alguien se lo quiere montar por su cuenta.
<!--more-->
{{< notice info >}}
**¿Que qué es qMp?** Visite http://qmp.cat/Inicio para conocerlo. 
Esta es la página de desarrollo proyecto https://dev.qmp.cat/projects/qmp  **🙌**
{{< /notice >}}
## Material necesario
### Antena
Puedes usar cualquier antena de la lista http://fw.qmp.cat/ que trabaje en 802.11a/ wifi 5GHz
Nosotros empezamos desplegando con NanoStation Loco M5 
![Ubiquiti NanoStation Loco M5](/gallery/red/2NSlocoM2.png)
Pero como cada vez hay más problemas para cargar el qMp ahora usamos como CPE las COMFAST CF-E120A
![COMFAST CF-E120A v3](/gallery/red/comfast_cf-120a.png)
http://en.comfast.com.cn/index.php?m=content&c=index&a=show&catid=19&id=23
https://openwrt.org/toh/comfast/cf-e120a
http://fw.qmp.cat/experimental/qMp_trunk-Kalimotxo_Comfast_CF-E120A-v3_sysupgrade_20210325-2110.bin

Las antenas se han comportado muy bien, aguantan frio y calor, hielo, agua, nieve... estamos a unos 1500 msnm y sin problema.

### Router neutro
Para tener señal dentro de casa sirve cualquier **router neutro**, ya va a depender de nuestras necesidades la capacidad del router.
![tp-link-tl-wr841n](/gallery/red/tp-link-tl-wr841n.png)
- Sirve cualquier router de operadoras reciclado con firmware OpenWrt. Lista de hardware compatible https://openwrt.org/toh/start 
- Hemos usado también los Next 3020 (que admiten qMp) https://openwrt.org/toh/nexx/wt3020
- Los Mikrotik hAP lite TC / RB941-2nD-TC https://mikrotik.com/product/RB941-2nD-TC
- Y los que más por calidad precio TP-Link TL-WR841N https://www.tp-link.com/es/home-networking/wifi-router/tl-wr841n/ (ojo con la versión)
- En el mercado de segunda mano siempre se pueden encontrar routers a buen precio.
### Herramientas
Las herramientas necesarias dependen de cada escenario, si vamos a poner el CPE en el tejado se puede usar el mástil de la antena de TV, también se pueden usar soportes de pared o de ventana con una ventosa. Hay que poner la antena en una posición en la que se vea físicamente con al menos otra antena de la red. Se tiene que trabajar con las herramientas adecuadas y con los materiales adecuados, para cableados expuestos a la intemperie hay que usar cable especifico que soporte intemperie, buena crimpadora, conectores RJ-45 fiables ... 
## Alta de nodo en la web guifi.net
### Registro
Lo primero hay que crearse un usuario en la web guifi.net en caso de que no lo tengamos ya.
### Crear el nodo
Vamos a la web guifi.net, disponibilidad, https://guifi.net/es/node/66713/view/availability y nos fijamos la última red que se esté usado, nos vamos a adueñar de la siguiente red (asegurando que esté libre y que no se esté usando ya en otro nodo).
![disponibilidad](/images/web_guifi/0guifidisponibilidad.png)
Si no tenemos habilidades de subneting vamos a una web de subneting y ponemos la ip. Por ejemplo  en http://jodies.de/
```
Address:   10.0.21.144           00001010.00000000.00010101.10010 000
Netmask:   255.255.255.248 = 29  11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7               00000000.00000000.00000000.00000 111
=>
Network:   10.0.21.144/29        00001010.00000000.00010101.10010 000 (Class A)
Broadcast: 10.0.21.151           00001010.00000000.00010101.10010 111
HostMin:   10.0.21.145           00001010.00000000.00010101.10010 001
HostMax:   10.0.21.150           00001010.00000000.00010101.10010 110
Hosts/Net: 6                     (Private Internet)
```
Vemos que la primera ip de la siguiente red es la ****10.0.21.145/29** esta será la ip de nuestra antena.**
Vamos a agregar el nodo, click en **Crear contenido** y se nos abre el menú de crear contenido, ahora click en **Nodo guifi.net.**
![Creando el nodo](/images/web_guifi/0guificrear_contenido.png)
Para poner el nombre de la ubicación seguimos este criterio:

**municipio-callenúmero-qmp**

![Creando el nodo](/images/web_guifi/1guifi_nodo.png)

Lo siguiente y mínimo indispensable es la ubicación, leemos y aceptamos la Licencia y acuerdo de utilización, el Pro-Común de la Red Abierta, Libre y Neutral "XOLN"  https://guifi.net/es/ProcomunXOLN, Vamos a la zona que nos corresponde 66713-Cepeda la Mora y en el mapa buscamos la ubicación en el mapa de nuestro nodo, hacemos click en la posición y se autocompletaran las coordenadas, también se puede poner una descripción del sitio y obligatoriamente hay que poner la elevación que tendrá la antena en metros, la distancia al suelo.
![Creando el nodo](/images/web_guifi/2guifi_nodo.png)
Hacemos click en GUARDAR **y ya está creado el nodo!**
Lo siguiente es agregar la antena al nodo.
Vamos a **Añadir nuevo dispositivo:** y añadimos un router.
![Creando el nodo](/images/web_guifi/3guifi_nodo.png)
En el router vamos a poner como nombre corto el mismo que pusimos en la ubicación del nodo **"cpd-Fe10-qmp"**
![Añadiendo el router](/images/web_guifi/4guifi_nodo.png)
Rellenamos la seccion **"Modelo de dispositivo, firmware y dirección MAC ()"** , **Wireless networking section** y **Parámetros de antena** Tal cual, con las mismas MAC, igual si usamos NanoStation Loco M5 ó Comfast
![Añadiendo el router](/images/web_guifi/5guifi_nodo.png)
Por último fijamos la ip y la red de la antena, la que habíamos previsto en el primer paso, dejo un comentario para acordarme que en realidad es Comfast.
![Añadiendo el router](/images/web_guifi/6guifi_nodo.png)
Click en guardar y salir y ya está, ya tenemos presencia en la web guifi.net
![Añadiendo el router](/images/web_guifi/7guifi_nodo.png)
Este es el nodo que hemos creado, https://guifi.net/es/node/129487 es para una amiga y espero ponerlo en producción muy pronto.
## Configuración de equipos
Una vez que tenemos el trasto con el firmware qMp configuramos lo primero el modo de trabajo, nosotros estamos en modo Community.
Asignamos los parámetros de configuración que tenemos en la web guifi.net
IP y red, SSID, Canal y codificación de país.
Con esto sería suficiente para que la antena de manera autónoma se conectara con la red y se pusiera a prestar servicio.
Es conveniente establecer hostname y SSID según web guifi.net.
Hay un otro camino para hacer la configuración de los equipos, una vez hemos creado el nodo y el equipo en la web guifi.net, se puede descargar y aplicar en en el router un archivo de configuración, es un script que nos asignará todos los parámetros de la web a nuestro equipo físico. Se llama **UNSOLOCLICK.** 
También configurar NTP y Cron con reinicio nocturno para evitar bloqueos.
Para el router de la wifi interior, la WAN es cliente de DHCP, lo conectaremos directamente al POE de la antena y la LAN será servidor de DHCP por wifi o cable.
Establecer password y hacer backup de las configuraciones de los equipos.
## Conectar a la red comunitaria
Una vez que los equipos están configurados si todo va bien, tendremos visibilidad de la toda la red y salida a internet en unos segundos. No hay que hacer nada más. Es routig es transparente, no necesita configuración a menos que se se quiera tomar un camino en concreto.
Antes de subir al tejado para hacer la instalación definitiva es importante tener la seguridad de que el equipo funciona. No cuesta más de 10 minutos hacer una instalación provisional para comprobar y nos ahorrará muchos quebraderos de cabeza y tiempo.

No hay un esquema de red fijo. Depende de las antenas que estén operativas. Un nodo se conecta con todos los nodos que que puede alcanzar y a su vez extiende la red dando cobertura, por eso en el mapa de Cepeda no salen las líneas de los enlaces, esto solo aparece en los enlaces punto a punto.

![topología de red mesh qmp](/gallery/red/nodos_qmp_cpd.png)

