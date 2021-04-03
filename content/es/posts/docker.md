---
title: "Docker"
date: 2017-05-30
draft: false
description: "Guía Docker"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
image: images/feature1/docker_logo.png
tags: 
- docker
- debian
---
Listar imagenes
```
#docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
```
borrar imagenes
```
#docker images
#docker rmi IMAGEN-ID
```
Borra todas la imagenes 
```
#docker rmi $(docker images -q)
```
Entrar en la consola de una imagen 
```
 docker run -t -i debian /bin/bash
```
listar contenedores
```
#docker ps -a
```
borra contenedor
```
#docker rm -f ad287f00940b
```
borra todos los contenedores
```
#docker stop $(docker ps -a -q)
#docker rm $(docker ps -a -q)
````
borra volúmenes huerfanos
```
#docker volume rm $(docker volume ls -qf dangling=true)
```
Cada vez que se lanza una imagen con docker run -t -i debian /bin/bash se crea un contenedor asociado, ver con: 
```
#docker ps -a
```
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
```
eb714a2b0b90 1b8740b73d63 "/bin/bash" 3 minutes ago Up 3 minutes jolly_spence
72eabf0db5d5 1b8740b73d63 "/bin/bash" 7 minutes ago Exited (0) 3 minutes ago affectionate_jang
dec292ad6ef4 debian "/bin/bash" 21 minutes ago Exited (0) 10 minutes ago suspicious_babbage
```
Para salir del contenedor en el que estamos trabajando y dejarlo funcionando lo hacemos con la teclas
**Ctrt+p+q**
Para volver a un contenedor que está corriendo
```
#docker attach CONTAINER ID
```
Para guardar los cambios en el contenedor donde hemos estado trabajando tecleamos
```
#docker commit CONTAINER ID
#docker commit dec292ad6ef4 sha256:1b8740b73d63b18d701b2964b8953e33635aeee39b0fa474981fffaf3b451d18
#docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
<none> <none> 1b8740b73d63 16 minutes ago 116MB
debian latest a2ff708b7413 8 days ago 100MB
```
para guardar una imagen de nuestro contenedor
```
# docker commit CONTAINER ID pull:TAG # docker commit dec292ad6ef4 jv4nj0/juanjo:test sha256:a5f831c9b55167935094995005c837866499d76ada585629735a28630f496caf
```
para guardar una imagen de nuestro contenedor en nuestro repositorio de docker https://hub.docker.com
```
#docker login

user /pass de la cuenta docker
#docker images
REPOSITORY              TAG IMAGE ID CREATED SIZE
xxxxx/yyyyy              test 82efda58c6f            12 seconds ago            116MB

xxxxx/yyyyy              latest 19c1de4a47d0      48 seconds ago           116MB

<none> <none>         a5f831c9b551                   31 minutes ago            116MB

<none> <none>         1b8740b73d63                About an hour ago      116MB

debian latest                a2ff708b7413                8 days ago                     100MB

#docker push IMAGE ID:TAG
#docker history imagen
```
entrega todas las modificaciones de la imagen
```
#dcoker inspect imagen
entrega info de configuracion de de la imagen
firma
cuando fue creado
imegen de la que depende.
hostname
puertos
gw
mac
```
Busca imagenes relacionadas
```
#docker search nembre de imagen
```
ALMACENAMIENTO
```
data volumen
Directorios del host montados en un directorio del docker
contenedores Data volumen
LOGS
docker logs id
RUN image
docker run -ti -d --name prueba IMAGEN
```
conectar a docker
```
docker exec -ti prueba /bin/bash
```
VOLUMENES
```
docker create
docker volume create NOMBRE
docker volume ls
docker volume rm
docker volume ls
```
ej
```
docker create -v /tmp --name datacontainer ubuntu
Network9000/#/
docker network connect/disconnect | create | inspect

* construir docker: dos vias docker file o docker compose evitar modificar contenedor y guardarlo como imagen
se usa un docker file para crear el contenedor
```
nano dockerfile
```
FROM nombre /nombre:version
MANTAINER xxxxxxx
RUN apt-get update

etc
```
despues para guardar
```
docker buid -t nombrenuevo/nombrenuevo:version ./
BRIDGE HOST NONE
interface equipo fisico solo loopback
docker0
```