---
title: "Escritorio remoto debian con x11vnc"
date: 2017-04-14
draft: false
description: "Instalación de servidor de escritorio remoto en Debian con x11vnc"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- debian
- fotos
categories:
- Linux
series:
- Red comunitaria
image: images/feature1/debian_logo.png
---
{{< notice info >}}
https://github.com/LibVNC/x11vnc
{{< /notice >}}
Instalamos servidor:
```
$ sudo apt-get update
$ sudo apt-get install x11vnc
```
Poner el password:
```
$ x11vnc -storepasswd
Enter VNC password: *********
Verify password: *********
Write password to /home/USUARIO/.vnc/passwd? [y]/n y
Password written to: /home/USUARIO/.vnc/passwd
```
Opciones para lanzar el servidor en
```
$ man x11vnc :
-forever : Mantiene el servidor en funcionamiento después de la sesion.
-rfbauth /home/USUARIO/.vnc/passwd : Para que pida el password de la ruta que queramos.
-usepw : pide el password default ~/.vnc/passwd
-rfbport 5900 : Indicamos el puerto de trabajo, el que queramos, si no ponemos nada es el :5900
-display :0 : Display independiente del puerto 0 1 2 3 4 5
-bg : Lanza el server en background, segundo plano.
-auth : indicamos el gestor de escritorio que queremos usar, si no lo sabemos lanzar: $ x11vnc -findauth
```
Opciones para arrancar el servidor x11vnc: - Entrar por SSH y lanzarlo para una vez o para que se quede abierto.
Ejemplo para conectarse usando cliente vinagre, remmina, tincvnc ... En el servidor ejecutamos:
```
# iptables -A INPUT -p tcp -s 0.0.0.0/0 --dport 5900 -j ACCEPT    # Abrimos el puerto si fuera necesario
x11vnc -findauth          # Buscamos la ruta del XAUTHORITY
XAUTHORITY=/var/lib/xdm/authdir/authfiles/A:0-w1exWB
```
Tiene que haber escritorio para tener escritorio remoto de algo...
```
apt-get install xfce4 xorg
```
si no arranca el entorno gráfico arrancar con startx comprobar con
```
echo $DISPLAY
localhost:10.0
```
en el caso de no tener  .Xauthority entrar un par de veces por SSH  $ ssh -X myname@somehost  y se generará
```
USUARIO@Cldy1:/home/USUARIO# xauth
xauth: file /USUARIO/.Xauthority does not exist
Using authority file /USUARIO/.Xauthority
xauth>
$xauth
Using authority file /home/pi/.Xauthority
```
empezamos,
```
x11vnc -bg -usepw -auth /var/lib/xdm/authdir/authfiles/A:0-w1exWB -display :0 # Arrancamos el servicio en display :0 que corresponde con puerto 5900
# netstat  -tulpn  # Comprobar si el servicio está arrancado en el puerto
```
Otro ejemplo
```
x11vnc -bg -usepw -auth /var/lib/xdm/authdir/authfiles/A:0-53cTfA -display :0
```
Ejemplo para conectarse con un navegador web https, con Java, (con OpenJRE funciona ok).
```
x11vnc -rfbport 5901 -http_ssl -svc -usepw -env FD_GEOM=1280x720 -loop -auth /var/lib/xdm/authdir/authfiles/A:0-53cTfA -display :0
```
Otros ejemplos:
```
x11vnc -rfbport 5901 -forever -http_ssl -svc -usepw -env FD_GEOM=1280x720 -auth guess -loop --display :0
x11vnc -rfbport 5901 -forever -http_ssl -svc -usepw -env FD_GEOM=1599x1196 -auth guess -loop --display :0 -o /var/log/x11vnc.log
x11vnc -rfbport 5901 -forever -http_ssl -svc -usepw -env FD_GEOM=1599x1196 -loop -auth /usr/bin/xauth -display :0
x11vnc -rfbport 33334 -http_ssl -svc -usepw -clip 1280x720+0+0 -scale 1280x720 -loop -auth /var/lib/xdm/authdir/authfiles/A:0-Da2eQG -display :0

Java SSL viewer URL:     https://ip:5901/   (single port)
Java SSL viewer URL:     http://ip:5801/
```
Comprobar si el servicio abre el puerto con:
```
# netstat -tulpn
```
Podemos ver el log en: /var/log/upstart/x11vnc.log