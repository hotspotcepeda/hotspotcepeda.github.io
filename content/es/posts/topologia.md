---
title: "Topología"
date: 2021-04-17
draft: false
description: "Topología de la red"
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
categories:
- Ayto
series:
- Red comunitaria
image: images/feature1/flowchart.png
---
Voy a intentar explicar como funciona la red en Cepeda, en resumidas cuentas es fácil, solo hay que asignar a los nodos ó antenas, las ips de la web guifi.net para que darle un poco de orden y la frecuencia de trabajo para que se entiendan, el canal y el país en el que está trabajando la red comunitaria.
Los enlaces los negocia el qMp en base a lo que tenga al alcance. Es una red con vida propia.
<!--more-->
{{< notice info >}}
**¿Que qué es qMp?** Visite http://qmp.cat/Inicio para conocerlo. 
Esta es la página de desarrollo proyecto https://dev.qmp.cat/projects/qmp  **🙌**
{{< /notice >}}
## Esquema de red
No hay un esquema de red fijo. Depende de las antenas que estén operativas. Un nodo se conecta con todos los nodos que que puede alcanzar y a su vez extiende la red dando cobertura, por eso en el mapa de Cepeda no salen las líneas de los enlaces, esto solo aparece en los enlaces punto a punto.
En la página de guifi.net lo normal es ver lineas verdes de un nodo a otro para ver los enlaces, pero con la red en mesh las antenas se conectan con todas las antenas que tienen disponibles. Se conectan todas con todas y llevan el tráfico por el camino más conveniente según protocolo de routing. Los tiempos de respuesta entre nodos son muy buenos, 1 ms. El routig es transparente, es dinámico y no hay que configurarlo de manera manual a no ser que queramos tomar un camino especial.

El qMp también está pensado para despliegues en caso de emergencia como catástrofes usando el modo roaming. 
Es autoconfigurable.
Nosotros en Cepeda estamos en modo community **los nodos se ven entre si** para poder compartir servicios.

Cuando agregamos un nuevo nodo a la red solo tenemos que configurar manualmente la frecuencia de trabajo, el canal. Para agregar un nodo de manera ordenada especificamos ip, mascara, frecuencia, país, hostname y SSID.

![topología de red mesh qmp](/gallery/red/nodos_qmp_cpd.png)

## IPs
La asignación de ip para los nodos se hace según https://guifi.net/es/node/66713/view/availability para cada nodo se están reservando 8 ips, 6 host, /29. Cuando se agrega un nodo vamos a la web guifi.net de Cepeda y asignamos la siguiente /29 siguiendo el orden, por defecto al agregar un nodo y solicitar ip se asigna una IP máscara /24 de 255 direcciones, esto se puede modificar a máscara /29 manualmente.
El direccionamiento que estamos usando es 10.0.20.0/23 para los nodos y 10.228.150.0/24 para los servidores y salida internet, según el direccionamiento guifi.net https://guifi.net/es/node/66713/view/ipv4