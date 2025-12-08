---
title: "Nube Privada Familiar con Nextcloud: Un Viaje hacia la Autogobernanza Digital"
date: 2025-10-04
draft: false
description: "Un Viaje hacia la Autogobernanza Digital"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
  - Let's Encrypt
  - NPM
  - Nextcloud
  - privacidad
  - proxmox
  - homelab
categories:
  - Seguridad
  - Homelab
image: images/feature1/nextcloud_logo.png
---

# Nube pivada con Nextcloud: Viaje hacia la atogobernanza dgital

Después de un proceso de investigación y configuración, ¡finalmente he puesto en producción mi propia nube privada familiar basada en Nextcloud! Este proyecto no solo ha sido un viaje entretenido a nivel técnico, sino también un paso consciente hacia una mayor privacidad y control sobre nuestros datos digitales.

## ¿Por qué una Nube Privada?

En la era digital actual, la mayoría de nuestras fotos y documentos importantes residen en servicios en la nube de terceros. Si bien estos servicios ofrecen comodidad, también plantean preguntas sobre la privacidad, la seguridad y la propiedad de nuestros datos. Mi objetivo principal era tener una solución de almacenamiento:

*   **Propia:** Con control total sobre la infraestructura y los datos.
*   **Segura:** Protegida contra accesos no autorizados y posibles brechas de seguridad.
*   **Estable y Duradera:** Una plataforma fiable que sirva a mi familia durante muchos años.
*   **Privada:** Asegurando que nuestros recuerdos y documentos personales permanezcan solo nuestros.

## La Pila Tecnológica: Nextcloud, Proxmox y Nginx Proxy Manager

La implementación se basa en una combinación robusta de tecnologías de código abierto:

1.  **Proxmox VE:** El hipervisor de virtualización. Nextcloud se ejecuta como una máquina virtual (VM) invitada sobre Proxmox. Esto ofrece una gran flexibilidad para gestionar recursos, realizar copias de seguridad y, si fuera necesario, migrar la VM.

2.  **Nextcloud:** El corazón de la nube. Nextcloud es una suite de productividad autoalojada que ofrece almacenamiento de archivos, sincronización, uso compartido y una gran cantidad de aplicaciones adicionales. Para el uso familiar, la aplicación "Memorias" es fantástica para organizar y visualizar fotos, mientras que la funcionalidad básica de almacenamiento y compartición de archivos cubre el resto de las necesidades.

3.  **Nginx Proxy Manager (NPM):** Un proxy inverso con interfaz gráfica que facilita la gestión de certificados SSL/TLS y el enrutamiento de solicitudes. Aunque el servidor Nextcloud no está directamente expuesto a Internet, NPM es crucial para gestionar el subdominio y los certificados Let's Encrypt dentro de la red local.

    *   **Subdominio:** Mi instancia de Nextcloud es accesible a través de `https://nextcloud.dominio.tal/`.
    *   **Certificados Let's Encrypt:** Gestionados automáticamente por NPM, estos certificados garantizan una conexión HTTPS segura y cifrada, lo cual es fundamental incluso dentro de una red local.

## Acceso Seguro: Tailscale VPN

Uno de los pilares de la seguridad de esta implementación es que el servidor Nextcloud **no es directamente accesible desde Internet**. El acceso se realiza exclusivamente a través de una Red Privada Virtual (VPN) utilizando **Tailscale**.

*   **¿Qué es Tailscale?** Es una VPN mesh que utiliza el protocolo WireGuard. Simplifica enormemente la configuración de una VPN al no requerir configuraciones de puertos o routers complejos. Crea una red segura entre todos los dispositivos autorizados, estén donde estén.
*   **Beneficios:** Esto significa que, ya sea que esté en casa o fuera, mi familia puede acceder a la nube como si estuviera en la red local, con toda la seguridad que ofrece WireGuard, sin exponer el servidor Nextcloud al público de Internet. Todos los datos viajan cifrados de extremo a extremo a través de la red de Tailscale.

## Consideraciones Técnicas Adicionales

*   **Hardware:** El servidor Proxmox se ejecuta en un hardware dedicado (ej. un mini PC, un servidor doméstico reciclado) que ofrece la capacidad de almacenamiento y RAM adecuadas para la VM de Nextcloud.
*   **Almacenamiento:** Para las fotos y archivos familiares, es crucial tener un almacenamiento redundante. Esto podría ser un RAID de software o hardware en el servidor Proxmox, o un sistema de copias de seguridad robusto que replique los datos a otra ubicación.
*   **Mantenimiento:**
    *   **Actualizaciones:** Es vital mantener tanto Nextcloud como el sistema operativo subyacente de la VM actualizados para garantizar la seguridad y el rendimiento.
    *   **Copias de Seguridad:** Implementar una estrategia de copias de seguridad regular de la VM de Nextcloud (incluyendo la base de datos y los archivos) es **imprescindible** para proteger los datos contra fallos de hardware o corrupción de datos. Proxmox facilita las copias de seguridad de las VMs.
    *   **Monitorización:** Aunque sea una instalación doméstica, una monitorización básica (ej. espacio en disco, uso de RAM) puede ayudar a anticipar problemas.

## Conclusión

Este proyecto de Nextcloud ha sido una experiencia muy gratificante. Tener el control total sobre nuestra nube privada nos proporciona tranquilidad y la certeza de que nuestros datos están seguros y privados. La combinación de Proxmox para la virtualización, Nextcloud como la plataforma de nube y Tailscale para el acceso seguro y privado, crea una solución potente y fácil de mantener.

Animo a cualquiera que valore su privacidad digital a explorar el camino de la autogobernanza. ¡Es un viaje que vale la pena!

---
