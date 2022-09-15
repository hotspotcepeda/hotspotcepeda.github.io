---
title: "Librenms VM"
date: 2022-09-15
draft: false
description: "Librenms VM"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- Ubuntu
- Linux
- Proxmox
- Librenms
categories:
- Ayto
series:
- Red comunitaria
libraries:
- mermaid
image: images/feature1/librenms.png
---
Como al actulizar me he cargado la maquina virtual de servidor de SNNMP Librenms al volver a instalar de 0 me ha costado un poco por que no me acordaba de nada.
Voy a instalar la misma que tenia, la 18.04 y voy a ir actualizando para ver si le cojo el punto a Ubuntu, nunca me habia cargado nada actualizando Debian.
<!--more-->
### Crear de nuevo la maquina virtual en proxmox
https://docs.librenms.org/Installation/Images/
La imagen en OVA de Librenms está disponible en:
https://github.com/librenms/packer-builds/releases?page=1
Para convertir el archivo de imagen .ova en VM Proxmox seguí los pasos de este tutorial:
https://www.itsfullofstars.de/2019/07/import-ova-as-proxmox-vm/
### Poner en marcha la RED
De primeras la imagen está lista para trabajar pero faltan algunas cosillas.
Default password:
```
librenms login: librenms
password: CDne3fwdfds
```
Para ponerse en modo root 
```
sudo su
```
Para cambiar el password:
```
passwd librems
```
Pero para configurar la red no es como en debian,
Lo primero es sacar el mombre de la interface en mi caso la ethernet... como no funciona ifconfig:
```
sudo systemctl start systemd-networkd
sudo systemctl enable systemd-networkd
networkctl list
networkctl status
```
Una vez que sabemos el nombre de la interface ethernet configuramos la ip en mi caso ip fija, el archivo de configuracion no está en /etc/network/interfaces está en el arhivo:
/etc/netplan/50-cloud-init.yaml
Tiene que quedar con este formato para ip fija:
```
network:
  version: 2
  renderer: networkd
  ethernets:
    ens19:
      dhcp4: no
      addresses: [192.168.1.10/24]
      gateway4: 192.168.1.254
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
Si lo colocas tal cual como te muestro no tendrás problemas, el truco es:
Deja 2 espacios por cada nivel. Por ejemplo, la linea que dice version: 2 tiene dos espacios al inicio. La que dice ens19: tiene 4
Deja un espacio entre los dos puntos y el valor. Por ejemplo, entre dhcp4: y no hay un espacio.

Luego aplicamos los cambios ejecutando el siguiente comando:
``` 
sudo netplan apply
```
### Otras
Ahora que ya hay salida a internet vamos a dar otros retoques:
Teclado en castellano ISO
```
sudo dpkg-reconfigure locales
```
nano
Con vi es que no tengo practica https://www.linuxtotal.com.mx/index.php?cont=info_admon_010

bash-completion
indispensable para mi
```
nano /etc/bash.bashrc
enable # bash completion in interactive shells
 if ! shopt -oq posix; then
   if [ -f /usr/share/bash-completion/bash_completion ]; then
     . /usr/share/bash-completion/bash_completion
   elif [ -f /etc/bash_completion ]; then
     . /etc/bash_completion
   fi
 fi
```

SSH
enable sshd
```
nano /etc/ssh/sshd_config
enable delete # lines
Port 2023
AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::
```

neofetch
Add neofetch end file .bashrc user
```
librenms@librenms:~$ sudo su
            .-/+oossssoo+/-.               root@librenms
        `:+ssssssssssssssssss+:`           -------------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 18.04.6 LTS x86_64
    .ossssssssssssssssssdMMMNysssso.       Host: KVM/QEMU (Standard PC (i440FX + PIIX, 1996) pc-i440fx-5.2)
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 4.15.0-96-generic
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 2 hours, 7 mins
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Packages: 727
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Shell: bash 4.4.20
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   CPU: Common KVM (1) @ 2.266GHz
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   GPU: Vendor 1234 Device 1111
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Memory: 221MiB / 1992MiB
+sssshhhyNMMNyssssssssssssyNMMMysssssss+
.ssssssssdMMMNhsssssssssshNMMMdssssssss.
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.
root@librenms:/
```

net-tools
arp se usa para manipular la caché ARP del núcleo, usualmente para añadir o borrar una entrada o volcar la caché completa.
dnsdomainname muestra el nombre del dominio DNS del sistema.
domainname muestra o establece el nombre del dominio NIS/YP del sistema.
hostname muestra o establece el nombre del sistema actual.
ifconfig es la utilidad principal usada para configurar las interfaces de red.
nameif nombra interfaces de red basándose en las direcciones MAC.
netstat se usa para mostrar las conexiones de red, tablas de encaminamiento y estadísticas de las interfaces.
nisdomainname hace lo mismo que domainname.
plipconfig se usa para afinar los parámetros del dispositivo PLIP, para mejorar su rendimiento.
rarp se usa para manipular la tabla RARP del núcleo.
route se usa para manipular la tabla de encaminamiento IP.
slattach conecta una interfaz de red a una línea serie. Esto permite usar líneas de terminales normales para crear enlaces punto a punto con otras computadoras.

NTP
```
sudo dpkg-reconfigure tzdata
root@librenms:~# timedatectl
                      Local time: Thu 2022-09-15 19:05:03 CEST
                  Universal time: Thu 2022-09-15 17:05:03 UTC
                        RTC time: Thu 2022-09-15 17:05:03
                       Time zone: Europe/Madrid (CEST, +0200)
       System clock synchronized: no
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
root@librenms:~#
```

para instalar todo esto
```
apt update && sudo apt upgrade
apt install net-tools neofetch bash-completion nano
```

## BACKUP VM
Y llegados a este punto, a respaldar la VM para no perder todo el trabajo.
### Actualizar Ubuntu
```
root@librenms:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.6 LTS
Release:        18.04
Codename:       bionic
root@librenms:~#
```
La quiero acutualizar 18.04 "Bionic Beavera" la siguiente LTS la 20.04 "Focal Fossa" con soporte hasta 2025
https://www.fosslinux.com/38303/how-to-upgrade-to-ubuntu-20-04-lts-focal-fossa.htm
Esta vez que tenía hecho el backup ha funcionado todo perfectamente siguiendo las indicaciones estasy por ssh.
```
root@librenms:/opt/librenms# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.5 LTS
Release:        20.04
Codename:       focal
root@librenms:/opt/librenms#
```
```
librenms@librenms:~$ sudo su
            .-/+oossssoo+/-.               root@librenms
        `:+ssssssssssssssssss+:`           -------------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 20.04.5 LTS x86_64
    .ossssssssssssssssssdMMMNysssso.       Host: KVM/QEMU (Standard PC (i440FX + PIIX, 1996) pc-i440fx-5.2)
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 5.4.0-125-generic
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 4 mins
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Packages: 795 (dpkg)
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Shell: bash 5.0.17
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Resolution: 1024x768
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   CPU: Common KVM (1) @ 2.266GHz
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   GPU: 00:02.0 Vendor 1234 Device 1111
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Memory: 178MiB / 1983MiB
.ssssssssdMMMNhsssssssssshNMMMdssssssss.
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.

root@librenms:/opt/librenms#
```

{{< notice info >}}
Me sigue gustando mas debian.
{{< /notice >}}
