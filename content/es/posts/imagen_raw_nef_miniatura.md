---
title: "VISTA EN MINIATURA DE ARCHIVOS DE IMAGEN NEF, RAW"
date: 2016-12-03
draft: false
description: "Para ver una foto en miniatura de los archivos raw y nef en el navegador"
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
Muy fácil, para tener la vista en miniatura de archivos de imagen NEF, RAW y demás en Nautilus primero instalamos el UFRaw, si es que no lo tenemos ya:
```
# apt-get update
# apt-get install ufraw
```
Ahora generamos este archivo /usr/share/thumbnailers/raw.thumbnailer
```
# nano /usr/share/thumbnailers/raw.thumbnailer
```
Copiamos este contenido en el archivo:
```
[Thumbnailer Entry]
Exec=/usr/bin/ufraw-batch --embedded-image --out-type=png --size=%s %u --overwrite --silent --output=%o
MimeType=image/x-3fr;image/x-adobe-dng;image/x-arw;image/x-bay;image/x-canon-cr2;image/x-canon-crw;image/x-cap;image/x-cr2;image/x-crw;image/x-dcr;image/x-dcraw;image/x-dcs;image/x-dng;image/x-drf;image/x-eip;image/x-erf;image/x-fff;image/x-fuji-raf;image/x-iiq;image/x-k25;image/x-kdc;image/x-mef;image/x-minolta-mrw;image/x-mos;image/x-mrw;image/x-nef;image/x-nikon-nef;image/x-nrw;image/x-olympus-orf;image/x-orf;image/x-panasonic-raw;image/x-pef;image/x-pentax-pef;image/x-ptx;image/x-pxn;image/x-r3d;image/x-raf;image/x-raw;image/x-rw2;image/x-rwl;image/x-rwz;image/x-sigma-x3f;image/x-sony-arw;image/x-sony-sr2;image/x-sony-srf;image/x-sr2;image/x-srf;image/x-x3f;
```
Salvar el archivo y ya está.