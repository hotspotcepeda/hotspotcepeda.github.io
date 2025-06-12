---
title: "LXC en Proxmox"
date: 2024-12-06
draft: false
description: "Contenedores LXC en entorno Proxmox."
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
  - lxc
  - proxmox
  - contenedores
  - virtualización
  - devops
categories:
  - Homelab
  - Virtualización
image: images/feature1/lxc-proxmox-banner.png
---

## 🔍 ¿Qué es LXC ?

**LXC** (Linux Containers) es una tecnología de virtualización a nivel de sistema operativo que permite ejecutar múltiples sistemas Linux aislados, conocidos como contenedores (CTs), en un único host. A diferencia de las Máquinas Virtuales (VMs) tradicionales que emulan hardware completo, los contenedores LXC comparten el kernel del host.

Esto se traduce en eficiencia :

- **Arranque Instantáneo:** Los contenedores se inician en segundos.
- **Bajo Overhead:** Consumo mínimo de RAM y CPU, permitiendo una mayor densidad de servicios en el mismo hardware.
- **Integración Nativa:** Se sienten como una VM ligera, con su propia IP y servicios, pero sin el peso del hipervisor.

![Arquitectura LXC vs VM](images/feature1/lxc-vs-vm.png)

### Conceptos Clave en Proxmox

*   **Template (Plantilla):** Imágenes de sistema operativo preconfiguradas (Ubuntu, Debian, Alpine, etc.) que sirven como base para tus contenedores.
*   **Contenedor Privilegiado:** Tiene acceso casi total a los recursos del host. Más compatible, pero menos seguro.
*   **Contenedor No Privilegiado:** Mapea los UIDs/GIDs, aislando el contenedor del host. Es la opción recomendada por seguridad.
*   **Bind Mount:** Permite compartir un directorio del host de Proxmox directamente dentro de un contenedor. Ideal para datos compartidos.
*   **Integración con ZFS:** Si usas ZFS como almacenamiento, los snapshots y clones de contenedores son casi instantáneos y muy eficientes en espacio.

## 🛠️ Comandos `pct`: La Navaja Suiza para LXC en Proxmox

Olvídate de `lxc-*`. En Proxmox, el comando `pct` (Proxmox Container Toolkit) manda. Comandos esenciales .

*(Ejemplo:  `100` es el ID de tu contenedor)*

### 1. Gestión Básica del Ciclo de Vida

```bash
# Listar todos los contenedores en el nodo
pct list

# Crear un nuevo contenedor (CT) desde una plantilla descargada
pct create 100 local:vztmpl/ubuntu-22.04-standard.tar.zst \
  --hostname mi-super-ct --storage local-lvm --memory 1024 --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp --password 'TuContraseñaSegura'

# Iniciar, detener, y reiniciar un contenedor
pct start 100
pct stop 100
pct reboot 100

# Acceder a la consola del contenedor 
pct enter 100

# Clonar un contenedor existente
pct clone 100 101 --hostname mi-clon

# Eliminar un contenedor (¡con cuidado!)
pct destroy 100
```

### 2. Configuración y Recursos

```bash
# Cambiar recursos (CPU y RAM) en caliente
pct set 100 --cores 4 --memory 2048

# Ver la configuración completa de un CT
pct config 100

# Configurar una red estática
pct set 100 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.100/24,gw=192.168.1.1

# Añadir un "bind mount" para compartir datos
# Sintaxis: --mp[id] /ruta/en/host,mp=/ruta/en/contenedor
pct set 100 --mp0 /mnt/data/media,mp=/media

# Aumentar el tamaño del disco raíz (ej. +5GB)
pct resize 100 rootfs +5G
```

### 3. Snapshots y Backups

```bash
# Crear un snapshot con descripción
pct snapshot 100 pre-actualizacion --description "Antes de instalar Nginx"

# Listar todos los snapshots de un CT
pct listsnapshot 100

# Revertir a un snapshot anterior (¡acción destructiva!)
pct rollback 100 pre-actualizacion

# Crear un backup completo y comprimido
vzdump 100 --compress zstd --storage mi-storage-de-backups --mode snapshot
```

## 🧠 Buenas Prácticas 

1.  **Seguridad Primero:** Usa siempre **contenedores no privilegiados** (`--unprivileged 1` al crear) a menos que una aplicación lo requiera específicamente (ej. Docker dentro de LXC).
2.  **Limita los Recursos:** No asignes más CPU o RAM de la necesaria. Usa `--cpulimit` y `--cpuunits` para controlar el rendimiento.
3.  **Usar `pct exec` para Comandos Rápidos:** En lugar de entrar a la consola, puedes ejecutar comandos directamente desde el host. Es ideal para scripts de automatización.
    ```bash
    # Actualizar un contenedor sin entrar en él
    pct exec 100 -- apt update && apt upgrade -y
    ```
4.  **Aprovecha ZFS:** Si tienes la opción, usa ZFS para tu almacenamiento de contenedores. Los snapshots y clones son una maravilla.
5.  **Plantillas Personalizadas:** Una vez que tengas un contenedor base configurado a tu gusto (con tus usuarios, paquetes, etc.), crea una plantilla a partir de él para desplegar nuevos servicios en segundos.
    ```bash
    # (Desde la GUI de Proxmox) Convertir CT a Plantilla
    pct set 100 --features nesting=1,fuse=1 --hookscript local:snippets/hookscript.pl
    ```

## 📊 Comparativa Rápida: LXC vs. Docker vs. VM

| Característica      | LXC (en Proxmox)                | Docker                       | VM Tradicional            |
| ------------------- | ------------------------------- | ---------------------------- | ------------------------- |
| **Aislamiento**     | Nivel de SO (medio-alto)        | Nivel de Proceso (bajo)      | Nivel de Hardware (muy alto) |
| **Overhead**        | Mínimo                          | Mínimo                       | Alto (RAM, CPU, Disco)    |
| **Arranque**        | Instantáneo (< 3 seg)           | Instantáneo (< 1 seg)        | Lento (segundos a minutos) |
| **Gestión**         | Sistema completo (como una VM)  | Aplicación única y efímera   | Sistema completo          |
| **Caso de Uso Ideal** | Servicios de infraestructura (DNS, VPN, DBs) | Microservicios, aplicaciones web | Sistemas operativos diferentes (Windows, BSD) |

## 🎯 Casos de Uso Perfectos para LXC

*   **Servidores Web y Reverse Proxies:** Nginx, Apache, Caddy, Nginx Proxy Manager.
*   **Servicios de Red:** AdGuard Home, Pi-hole, Unbound, WireGuard.
*   **Bases de Datos Ligeras:** SQLite, PostgreSQL, MariaDB para proyectos pequeños/medianos.
*   **Bots y Scripts de Automatización:** Contenedores dedicados para bots de Discord, Telegram, o scripts de Python.
*   **Entornos de Desarrollo:** Un contenedor por proyecto para aislar dependencias sin el peso de Docker.

 ¡ Al ataque ! 🚀
````