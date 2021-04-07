---
title: "Portal cautivo hotspot mikrotik"
date: 2016-03-09
draft: false
description: "Portal cautivo y la redirección de https"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- hotspot
- wifi
- cepeda la mora
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
{{< notice info >}}
Vista la experiencia no soy partidario de usar hotspot con portal cautivo, entiendo que en algunos escenarios puede ser de utilidad.
{{< /notice >}}
## Intro
La cosa es que cuando hacemos un hotspot con MikroTik una de los problemas que nos encontramos tras hacer el wizard es que el usuario que se conecta obtiene ip de manera correcta pero a la hora de ser redireccionado al portal cautivo si va a una dirección https no se le muestra la página y le sale un error 404.
Y es que dentro de la Router Board hay un servidor web donde tenemos que hacer nuestra página web.
![Portal cautivo modelo ](/gallery/red/hotspot.PNG)
Para solucionar la redireecion con https hay que generar un par de archivos e importarlos en el Mikrotik, ya sea por consola o por winbox. **Necesitamos un certificado y una clave**, .crt y .key, para generarlos necesitamos la herramienta openssl, si no la tenemos lo primero seria instalarlos, bueno lo primero sería instalar una distribución de Linux.
```
# apt-get update
# apt-get install openssl
```
## Clave privada
Generamos una clave privada y una key, nos pedirá un password que debemos recordar.
```
# openssl genrsa -des3 -out ca.key 4096
 Enter pass phrase for ca.key: (password)
```
Ya tenemos un archivo que se llama ca.key, 
## Certificado
Ahora vamos a generar el .crt con esa key en este caso es para 3650 días, 10 años.
```
# openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```
Empieza el dialogo donde nos va preguntando datos y vamos poniendo lo que nos pide.
Ahora generamos la key/certificado par para el servidor, nos pide de nuevo contraseña, recordar:
```
 # openssl genrsa -des3 -out server.key 4096
 Generating RSA private key, 4096 bit long modulus
 ...........................................................................................................................++
 ....................................++
 e is 65537 (0x10001)
 Enter pass phrase for server.key: (passwor)
 Verifying - Enter pass phrase for server.key: (password)
```
Se asocia la key al certificado y de nuevo el dialogo donde vamos poniendo lo que nos pida:
```
 # openssl req -new -key server.key -out server.csr
 Enter pass phrase for server.key: (password)
 You are about to be asked to enter information that will be incorporated
 into your certificate request.
 What you are about to enter is what is called a Distinguished Name or a DN.
 There are quite a few fields but you can leave some blank
 For some fields there will be a default value,
 If you enter '.', the field will be left blank.
 -----
 Country Name (2 letter code) [AU]:
 State or Province Name (full name) [Some-State]:
 Locality Name (eg, city) []:
 Organization Name (eg, company) [Internet Widgits Pty Ltd]:
 Organizational Unit Name (eg, section) []:
 Common Name (e.g. server FQDN or YOUR name) []:
 Email Address []:
Please enter the following 'extra' attributes
 to be sent with your certificate request
 A challenge password []:(password)
 An optional company name []:
```
## Asociar los crs y la key al crt 
Ejecutamos:
```
 # openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
 Signature ok
 subject=/C=  /ST=  /L=  /O=  /OU=  /CN=  /emailAddress=
 Getting CA Private Key
 Enter pass phrase for ca.key: (password)
```
Estos son los archivos que nos quedan al finalizar si hay algo raro o que no os guste, pues se repiten todos los pasos.
```
# ls
 ca.crt    ca.key    server.crt  server.csr    server.key
```
## Importar a la MikroTik
Luego para importarlos solo tenemos que copiar 2 archivos, el server.crt y server.key en el MikroTik, los otros archivos los guardamos. Por ftp o por WinBox los pasamos a files y los importamos por consola o desde el menu SYSTEM->CERTIFICATES del WinBox. Este es el ejemplo de importación desde la consola del MikroTik.
```
[admin@MikroTik] /certificate> import file-name=server.crt
 passphrase:(password)
 certificates-imported: 1
 private-keys-imported: 0
 files-imported: 1
 decryption-failures: 0
 keys-with-no-certificate: 0
 [admin@MikroTik] /certificate> import file-name=server.key
 passphrase:(password)
 certificates-imported: 0
 private-keys-imported: 1
 files-imported: 1
 decryption-failures: 0
 keys-with-no-certificate: 0
```
## Configurar en hotspot
Ya solo nos queda marcar en la casilla del hotspot -> «login by» **https** y en services el https seleccionando el certificado que hemos importado.
Ahora cuando se conecte al hotspot y se tenga una página https como pagína de inicio en su navegador (que es lo más normal), se le redireccionará al portal cautivo donde se presentará al HotSpot y ya sea con usuario y contraseña o como invitado, podrá navegar y hacer uso del servicio.
Este es el portal cautivo que teníamos en Cepeda.
![Portal cautivo Cepeda ](/gallery/red/hotspot_cepeda.PNG)