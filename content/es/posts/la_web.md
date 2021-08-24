---
title: "La Web"
date: 2021-08-24
draft: false
description: "La Web"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- github
- Visual Studio Code
- markdown
- cepeda la mora
- HUGO
- hotspotcepeda
categories:
- Ayto
series:
- Red comunitaria
libraries:
- mermaid
image: images/feature1/hotspot.png
---
Voy a intentar explicar como he hecho esta web. En la asociación quieren tener una web y creo que de esta manera se puede hacer bien, rápido, fiable, colaborativa, segura ...  
<!--more-->
## Ingredientes
### HUGO
https://gohugo.io/
Como generador de sitio web estático escrito en Go, básicamente escribes los contenidos en Markdown (lenguaje humano) y HUGO lo transforma en HTML y CCS
```
hugo server --> Crea un servidor local con tu web en http://localhost:1313/ es LIVE.
hugo - d docs/ --> Vuelca tu contenido a HTML CCS.
```
### GitHub
https://github.com/hotspotcepeda/hotspotcepeda.github.io
Como controlador de versiones en nube.

<object data="/pdfs/comandos_git.pdf#page=1" type="application/pdf" width="100%" height="950px">
   <p>Chuleta con comandos y resumen git <a href="/pdfs/comandos_git.pdf">aquí</a></p>  
</object>

```
git clone https://github.com/hotspotcepeda/cepeda.git --> Clona a local el repositorio
git pull --> Compara diferencias entre local y en repositorio
git add . --> marca para subir a repositorio los cambios
git commit -m "mensaje para control de versiones"
git push --> Sube los cambios marcados de local a repositorio
```
### Google
Una cuenta de correo google o el correo que sea va ha hacer falta, para logarse en github, para hacer el posicionamiento, de contacto del sitio, etc...
### Visual Code Studio
https://code.visualstudio.com/
Esto es lo mejor para perderle el respeto a la programación, Cuando está instalado y configurado se pueden hacer los post en 1 minuto, y muchas cosas más.

## Aprendizaje

Yo lo que hice fue pillar una plantilla de HUGO, descargarla a una carpeta de mi máquina, trabajarla con el Visual Code, subirla a github e ir modificando con mi contenido. 
Que te equivocas y te los cargas todo, pues no pasa nada, pasito para atrás y a probar de nuevo, es lo bueno de git.

Estos videos me vinieron muy bien para empezar.
- HUGO
https://www.youtube.com/playlist?list=PLIcuwIrm4rKe5VEaiD59Ix3AklqYxiG2f
- Git
https://www.youtube.com/playlist?list=PLIcuwIrm4rKddeIBeA_8HrXE27ZaDZoQE
- Visual Studio Code
https://www.youtube.com/watch?v=HVzFLw5r2EM
https://www.youtube.com/watch?v=AYbgqmyg7dk







{{< notice info >}}
La página también tiene el Brave Rewards para donaciones en BAT.
El posicionamiento de google y microsoft, Google Analytics...
{{< /notice >}}
