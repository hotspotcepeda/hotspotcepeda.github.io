---
title: "La propuesta"
date: 2014-06-03
draft: false
description: "La Propuesta que se hizo al Ayto"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
categories:
- Ayto
series:
- Red comunitaria
tags: 
- guifi.net
- qMp
- guifi.net
- Cepeda la Mora
image: images/feature2/mathbook.png
---

Me decidí a formular una propuesta para el despliegue de la red en Cepeda, como no tenia ni idea, le presenté al alcalde de aquella época [Javier Garcia](https://www.blogger.com/profile/13080724491994019511 "Blog Javier Garcia") un pdf, a lo que el alcalde respondó:

*"Mira, a mi no me vengas con cosas técnicas, si quieres hacerlo... adelante. Te voy a ayudar en lo que pueda."*
## Idea inicial
Está era la ida que tenía para el despliegue en 2014:
Una salida balanceada a internet con una RB-750 con un superservidor proxy para cachear a todo el que se asomara por allí y otra RB-750 para gestionar las conexiones del hotspot, ahí lo pone, 30 minutos al día con un caudal de 256 Kb en bajada y 64 Kb en subida. AP Unifi
Para conectar las viviendas un modelo en Infraestructura AP cliente con una antena central a la que se conectan las estaciones de las casas.
<object data="/pdfs/ESQUEMA DE RED1.pdf#page=1" type="application/pdf" width="100%" height="950px">
   Puedes descargar el archivo <a href="/pdfs/ESQUEMA DE RED1.pdf">aquí</a></p>  
</object>

## Propuesta
Este es el documento que se presentó al Ayto, en nombre de la AC RASPUTIN. No me extraña que el alcalde me dijera que hiciera lo que quisiera. 
El logo de la cabra es lo que más me gusta, está a punto de embestir a los ISP tradicionales.
<object data="/pdfs/CEPEDA_guifi.pdf#page=1" type="application/pdf" width="100%" height="950px">
  <p>Parece que no tienes plugin para visualizar PDF en tu navegador.
   Puedes descargar el archivo <a href="/pdfs/CEPEDA_guifi.pdf">aquí</a></p>  
</object>

***Lo que nos tenemos que tatuar es el procomún de la Red Abierta Libre y Neutral:***
El Procomún de la RALN se fundamenta en las siguientes bases:
1. Eres libre de utilizar la red para cualquier propósito mientras no perjudiques el
funcionamiento de la propia red, la libertad de otros usuarios, y respetes las
condiciones de los contenidos y servicios que circulan libremente.
2. Eres libre de conocer como es la red, sus componentes y su funcionamiento,
también puedes difundir su espíritu y funcionamiento libremente.
3. Eres libre para incorporar servicios y contenidos a la red con las condiciones
que quieras.
4. Eres libre de Incorporarte a la red y ayudar a extender estas libertades y
condiciones.

Es este documento está detallada la autoprovisión de nodos para el despliegue en modo infraestructura en la plataforma guifi.net
<object data="/pdfs/paso_a_paso.pdf#page=1" type="application/pdf" width="100%" height="950px">
  <p>Parece que no tienes plugin para visualizar PDF en tu navegador.
   Puedes descargar el archivo <a href="/pdfs/paso_a_paso.pdf">aquí</a></p>  
</object>

## Presupuesto
Se presentó un presupuesto en base a la propuesta. Creo que me salian 900 euros, esto es lo que se aprobó y lo que pagó el ayuntamiento más la salida a internet.
![Hoja de gastos Ayto Cepeda despliegue red comunitaria](/images/ayto/hoja_gastos_ayto.jpg)
600 euros para un ayuntamiento es un gasto ridículo, en las empresas que trabajan este producto el precio habitual para un proyecto así es de 6000 a 10000 euros llave en mano + el coste de la gestión y el mantenimiento de la red. ¿Lo ves caro? pues hazlo tú mismo.
Solo necesitas tener los conocimientos adecuados, bueno también te hará falta un ordenador y una conexión a internet para hacer la gestión remota, un espacio de trabajo, carnet de conducir y un coche grande o una furgoneta para poder desplazarte y transportar el equipamiento, los tramos de las torres y los mástiles, ropa de trabajo, EPIs, herramienta de mano, escalera grande de 3 tramos y pequeña, taladro, brocas, radial, discos, atornilladora, crimpadora, conectores, herrajes, abrazaderas, bridas, tacos, tornillos, cinta aislante, guiacables, aparatos de medida, comprobador de cables, cable ftp exterior, cable ftp interior, material eléctrico, conocimientos, negociaciones con proveedores, dinero para comprar el equipamiento, desplazamientos a proveedores, desplazamientos programados para labores preventivas, desplazamientos extraordinarios por averías, conocimientos, seguro de responsabilidad civil, titulación para trabajo en altura, tasas e impuestos, combustible, dinero, conocimientos y lo más valioso de todo, TU TIEMPO.
## SAX
Llegó el SAX 2015. El SAX (Salut, Amor i Xarxa), es un encuentro anual de la comunidad guifi.net, allí conocí al [Sr. Roger Pueyo](https://dev.qmp.cat/users/68 "Perfil de Roger Pueyo en dev.qmp.cat"), le comenté la idea que teníamos para desplegar en Cepeda y después de una charla muy amena, llegue a la conclusion de que lo más económico y productivo que podia hacer era desplegar la red en modo adhoc con [qMp](http://qmp.cat/Inicio "web qMp"), Quick Mesh Project.
Desplegar en modo infraestructura puede estár bien en algunos escenarios, si tienes una ubicación con alimentación eléctrica y acceso asegurado desde la que se domine todo el área que quieras iluminar, pues genial. Tendrás un punto central al que apuntar todas las conexiones, esto es una red centralizada punto-multipunto y si falla el punto central dejará de funcionar toda la red, puedes redundar la antena central para que esto no suceda. Cuando quieras conectar una nueva antena tendrás que configurar en los 2 extremos.
Pero en el escenario que tenemos en Cepeda no hay un sítio mágico desde el que se pueda ver todo el pueblo, estuve mirando el deposito de agua, pero eran todo problemas, la alimentación eléctrica, el acceso, la ubicación expuesta a robos y sabotajes, aparte que la salida a internet está en el ayuntamiento, tenía que hacer un enlace dedicado del Ayto al depósito con 2 equipos, 4 en caso de redundancia y otro equipo bien potente para conectar con todos los nodos.
## qMp
Con el despliegue en mesh la cobertura se va extendiendo con cada equipo que se incorpora a la red, se convierte en una red descentralizada que es un valor añadido.
Lo primero que hay que hacer en montar toda la red sobre una mesa, elije un espacio de trabajo y empieza a configurar equipos, ponlos en producción y sometelos durante un tiempo a un estricto seguimiento. Cuando tengas la seguridad de que los equipos funcionan como tú quieres los instalas, cuando estamos instalando no podemos hacer investigaciones ni pruebas ni otra cosa que no sea instalar, de está manera si falla algo en la instalación automáticamente se descartan problemas de configuración.
Para comenzar con el despliegue qMp recomiendo elegir ubicaciones elevadas y que siempre estén encendidas, de está manera la cobertura de la red se irá extendiendo sin esfuerzo a medida que se van agregando nuevos nodos.
Se puede desplegar en 2.4 GHZ 802.11bg o en 5 GHz 802.11a, dependiendo del escenario, si queremos desplegar en el interior de un edificio podemos hacerlo con equipos indoor y en 2.4 GHz, si los equipos están en la calle necesitamos CPE que soporten intemperie agua, hielo, sol y viento.
Hay que tener en cuenta que debemos elegir con mucha atención el hardware, tiene que ser compatible con OpenWRT/qMp. Esta es la lista de hardware compatible https://qmp.cat/Supported_devices
{{< notice warning >}}
- 27/03/2021 Ojo con las nuevas remesas de equipamiento Ubiquiti, las nuevas revisiones 2000K y superiores de NanoStation Loco M5 no funcionan ni con OpenWRT ar71xx ni con OpenWRT ath79, ni tan siquiera probando OpenWRT ath79 para Bullet M XW que sí funciona en las Rocket M XW. Al parecer hay dos arquitecturas distintas que sirven para distintas revisiones de hardware de Ubiquiti .La ar71xx es la de siempre, antes funcionaba, la ath79 es la nueva. No sé sabe todavía la manera de flashearlas. 
{{< /notice >}}




<!--more-->
---
