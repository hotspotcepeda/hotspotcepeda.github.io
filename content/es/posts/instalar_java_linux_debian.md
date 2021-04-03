---
title: "Instalación de Java en Linux Debian"
date: 2017-04-03
draft: false
description: "Instalación de Java en Linux Debian 8"
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
## Unninstall
DESINSTALACION TOTAL de todo lo relacionado con Java, Sun, Oracle, OpenJDK, IcedTea plugins, GIJ
```
dpkg-query -W -f='${binary:Package}\n' | grep -E -e '^(ia32-)?(sun|oracle)-java' -e '^openjdk-' -e '^icedtea' -e '^(default|gcj)-j(re|dk)' -e '^gcj-(.*)-j(re|dk)' -e '^java-common' | xargs sudo apt-get -y remove
sudo apt-get -y autoremove
```
Purge config files:
```
dpkg -l | grep ^rc | awk '{print($2)}' | xargs sudo apt-get -y purge
```
Remove Java config and cache directory:
```
sudo bash -c 'ls -d /home/*/.java' | xargs sudo rm -rf
```
Remove manually installed JVMs:
```
sudo rm -rf /usr/lib/jvm/*
```
Remove Java entries, if there is still any, from the alternatives:
```
for g in ControlPanel java java_vm javaws jcontrol jexec keytool mozilla-javaplugin.so orbd pack200 policytool rmid rmiregistry servertool tnameserv unpack200 appletviewer apt extcheck HtmlConverter idlj jar jarsigner javac javadoc javah javap jconsole jdb jhat jinfo jmap jps jrunscript jsadebugd jstack jstat jstatd native2ascii rmic schemagen serialver wsgen wsimport xjc xulrunner-1.9-javaplugin.so; do sudo update-alternatives --remove-all $g; done
Search for possible remaining Java directories:
sudo updatedb
sudo locate -b '\pack200'
```
If the command above produces any output like /path/to/jre1.6.0_34/bin/pack200 remove the directory that is parent of bin, like 
```
this: sudo rm -rf /path/to/jre1.6.0_34.
```
## Install

OpenJRE
```
sudo apt-get install openjdk-7-jre
```
IcedTea Plugin
```
sudo apt-get install icedtea-plugin
```
Ver que java está activo:
```
# update-alternatives --config java
Existen 2 opciones para la alternativa java (que provee /usr/bin/java).

Selección Ruta Prioridad Estado
------------------------------------------------------------
0 /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java 1071 modo automático
* 1 /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java 1071 modo manual
2 /usr/lib/jvm/java-8-oracle/jre/bin/java 1001 modo manual

Pulse <Intro> para mantener el valor por omisión [*] o pulse un número de selección: 0
# java -version
java version "1.7.0_131"
OpenJDK Runtime Environment (IcedTea 2.6.9) (7u131-2.6.9-2~deb8u1)
OpenJDK 64-Bit Server VM (build 24.131-b00, mixed mode)
```
Cambiar prioridad
````
# update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/java-8-oracle/jre/bin/java" 1001
update-alternatives: utilizando /usr/lib/jvm/java-8-oracle/jre/bin/java para proveer /usr/bin/javac (javac) en modo automático
```
