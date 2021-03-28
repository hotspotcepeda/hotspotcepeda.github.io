---
title: "CERTIFICADOS OPENVNP"
date: 2016-04-16
draft: false
description: "Crear CERTIFICADOS OPENVNP en Debian"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- debian
- openVPN
categories:
- Linux
series:
- Red comunitaria
image: images/feature1/openVPN_LOGO.png
---
{{< notice info >}}
https://openvpn.net/index.php/open-source/documentation/howto.html
https://aula128.wordpress.com/2015/02/07/openvpn-instalacion-y-configuracion-del-servidor-en-linux-y-autentificacion-de-un-cliente-windows-7/
http://www.redeszone.net/redes/openvpn/
{{< /notice >}}
```
apt-get install easy-rsa
apt-get install openvpn
```
Copiar los DIRECTORIOS easy-rsa y sample-config-files a /etc/openvpn/
```
/usr/share/doc/openvpn/examples# cp -R sample-config-files/ /etc/openvpn/
/usr/share# cp -R easy-rsa/ /etc/openvpn/
/etc/openvpn# ls
 easy-rsa  sample-config-files  update-resolv-conf
/etc/openvpn# cd sample-config-files/
/etc/openvpn/sample-config-files# ls
 client.conf  home.up          loopback-server  openvpn-shutdown.sh  README        static-home.conf    tls-home.conf     xinetd-client-config
 firewall.sh  loopback-client  office.up        openvpn-startup.sh   server.conf.gz  static-office.conf    tls-office.conf  xinetd-server-config
```
Crear carpeta de claves
```
/etc/openvpn/easy-rsa# mkdir keys
 /etc/openvpn/easy-rsa# ls
 build-ca     build-key           build-key-server  changelog.Debian.gz  inherit-inter  openssl-0.9.6.cnf    pkitool        revoke-full  whichopensslcnf
 build-dh     build-key-pass    build-req     clean-all          keys         openssl-0.9.8.cnf    README-2.0.gz  sign-req
 build-inter  build-key-pkcs12  build-req-pass     copyright          list-crl         openssl-1.0.0.cnf    README.Debian  vars
```
Editamos vars y modificamos KEY_COUNTRY, KEY_PROVINCE, KEY_CITY, KEY_ORG y KEY_MAIL… a nuestro gusto, También modificamos la ruta de salida de claves  export KEY_DIR=”/etc/openvpn/easy-rsa/keys/” y guardamos cambios.

Los parámetros Diffie Hellman se  generar para el servidor. Si se genera en una maquina potente tarda unos segundos, si lo generamos en un router domestico más de 40 minutos.
```
/etc/openvpn/easy-rsa# ./build-dh
```
Certificado para la CA
```
/etc/openvpn/easy-rsa# ./pkitool --initca
```
Certificado y llaves para el SERVER
```
/etc/openvpn/easy-rsa# ./pkitool --server servidor
```
Certificado y llaves para el CLIENTE
```
/etc/openvpn/easy-rsa# ./pkitool clienteA
/etc/openvpn/easy-rsa# ./pkitool clienteB
/etc/openvpn/easy-rsa# ./pkitool clienteX
```
Llave para servidor y cliente TLS-AUTH; Las directivas tls-auth ofrecen un nivel adicional de seguridad por encima de la proporcionada por SSL / TLS
```
/etc/openvpn/easy-rsa/keys# openvpn --genkey --secret sanpe.key
```
Se coloca en el mismo directorio que el .key RSA y archivos .crt.

En la configuración del servidor, añadir:
```
TLS-auth sanpe.key 0
```
En la configuración del cliente, añadir:
```
TLS-auth sanpe.key 1
```
Nuevo contenido de la carpeta keys.
```
/etc/openvpn/easy-rsa/keys# ls -sal
 total 336
 4 drwx------ 2 root root 4096 abr 16 14:55 .
 4 drwxr-xr-x 3 root root 4096 abr 16 14:55 ..
 8 -rw-r--r-- 1 root root 5715 abr 16 14:41 01.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:48 02.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 03.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 04.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 05.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 06.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 07.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 08.pem
 8 -rw-r--r-- 1 root root 5593 abr 16 14:51 09.pem
 8 -rw-r--r-- 1 root root 5594 abr 16 14:51 0A.pem
 8 -rw-r--r-- 1 root root 5594 abr 16 14:51 0B.pem
 8 -rw-r--r-- 1 root root 5594 abr 16 14:51 0C.pem
 4 -rw-r--r-- 1 root root 1801 abr 16 14:40 ca.crt
 4 -rw------- 1 root root 1704 abr 16 14:40 ca.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:48 clienteA.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:48 clienteA.csr
 4 -rw------- 1 root root 1708 abr 16 14:48 clienteA.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 clienteB.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:50 clienteB.csr
 4 -rw------- 1 root root 1704 abr 16 14:50 clienteB.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 clienteC.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:50 clienteC.csr
 4 -rw------- 1 root root 1704 abr 16 14:50 clienteC.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 clienteD.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:50 clienteD.csr
 4 -rw------- 1 root root 1704 abr 16 14:50 clienteD.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 clienteE.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:50 clienteE.csr
 4 -rw------- 1 root root 1704 abr 16 14:50 clienteE.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 clienteF.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:50 clienteF.csr
 4 -rw------- 1 root root 1704 abr 16 14:50 clienteF.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:50 clienteG.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:50 clienteG.csr
 4 -rw------- 1 root root 1704 abr 16 14:50 clienteG.key
 8 -rw-r--r-- 1 root root 5593 abr 16 14:51 clienteH.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:51 clienteH.csr
 4 -rw------- 1 root root 1704 abr 16 14:51 clienteH.key
 8 -rw-r--r-- 1 root root 5594 abr 16 14:51 clienteI.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:51 clienteI.csr
 4 -rw------- 1 root root 1704 abr 16 14:51 clienteI.key
 8 -rw-r--r-- 1 root root 5594 abr 16 14:51 clienteJ.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:51 clienteJ.csr
 4 -rw------- 1 root root 1708 abr 16 14:51 clienteJ.key
 8 -rw-r--r-- 1 root root 5594 abr 16 14:51 clienteK.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:51 clienteK.csr
 4 -rw------- 1 root root 1704 abr 16 14:51 clienteK.key
 4 -rw-r--r-- 1 root root 424 abr 16 14:38 dh2048.pem
 4 -rw-r--r-- 1 root root 1800 abr 16 14:51 index.txt
 4 -rw-r--r-- 1 root root 21 abr 16 14:51 index.txt.attr
 4 -rw-r--r-- 1 root root 21 abr 16 14:51 index.txt.attr.old
 4 -rw-r--r-- 1 root root 1650 abr 16 14:51 index.txt.old
 4 -rw------- 1 root root 636 abr 16 14:55 sanpe.key
 4 -rw-r--r-- 1 root root 3 abr 16 14:51 serial
 4 -rw-r--r-- 1 root root 3 abr 16 14:51 serial.old
 8 -rw-r--r-- 1 root root 5715 abr 16 14:41 servidor.crt
 4 -rw-r--r-- 1 root root 1102 abr 16 14:41 servidor.csr
 4 -rw------- 1 root root 1704 abr 16 14:41 servidor.key
```
El paso final en el proceso de generación de claves es copiar todos los archivos en las máquinas que los necesitan, teniendo cuidado de copiar los archivos secretos por un canal seguro.

![OpenVPN Files](/gallery/red/opeVPN_files.PNG)