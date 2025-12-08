---
title: "PBS recuperacion"
date: 2024-11-15
draft: false
description: "PBS_recuperacion de fallo catastrófico"
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
image: images/feature2/proxmox2.png
---

```markdown
# Recuperación de un Datastore de Proxmox Backup Server (PBS) tras Fallo del Hardware/SO

En el mundo de los respaldos, la capacidad de recuperar tus datos es tan importante como la capacidad de crearlos. Proxmox Backup Server (PBS) está diseñado para la resiliencia, permitiendo que tus Datastores sean portátiles y recuperables incluso si el servidor PBS original falla por completo (ya sea por un problema de hardware o una corrupción del sistema operativo).

Este post detalla los pasos y comandos esenciales para recuperar un Datastore de PBS montado en un disco externo USB en una máquina PBS completamente nueva.

---

## Escenario de Catástrofe

Imagina la siguiente situación:
*   Tu servidor PBS original (por ejemplo, un ThinkPad T430s) ha dejado de funcionar.
*   El Datastore de PBS reside en un disco externo USB (ej., `/dev/sdc` montado en `/mnt/backup_hdd`).
*   Necesitas acceder a esos respaldos lo antes posible.

---

## Proceso de Recuperación Paso a Paso

El proceso se divide en dos fases principales: **Preparación del Disco en el Nuevo Sistema** y **Configuración del Datastore en la Nueva Instancia de PBS**.

### Fase 1: Preparación del Disco en el Nuevo Sistema

Esta fase se realiza en la terminal de la nueva máquina que ejecutará Proxmox Backup Server. Asumimos que la nueva máquina ya tiene un sistema operativo compatible con PBS (Debian/Ubuntu Server) y que PBS está instalado.

1.  **Conecta el Disco Externo USB**
    Conecta físicamente tu disco externo USB (que contiene el Datastore de PBS) a la nueva máquina.

2.  **Identifica el Disco y la Partición**
    Es crucial identificar correctamente el nuevo nombre del dispositivo para tu disco de respaldo. Puede que ya no sea `/dev/sdc`. Usa `lsblk` para listar todos los dispositivos de bloque:

    ```bash
    lsblk
    ```
    Busca tu disco por su tamaño y particiones. Por ejemplo, si tu disco de 8TB se muestra como `/dev/sdb` y la partición principal es `/dev/sdb1`, ese será tu objetivo.

3.  **Crea un Punto de Montaje**
    Necesitas un directorio donde montar la partición del Datastore. Es buena práctica usar un nombre descriptivo, similar a tu configuración anterior.

    ```bash
    mkdir /mnt/backup_hdd
    ```
    *(Ajusta `/mnt/backup_hdd` según tu preferencia o configuración original).*

4.  **Monta la Partición del Datastore**
    Monta la partición identificada en el punto de montaje que acabas de crear.

    ```bash
    mount /dev/sdb1 /mnt/backup_hdd
    ```
    *(Reemplaza `/dev/sdb1` con el nombre de tu partición y `/mnt/backup_hdd` con tu punto de montaje).*

5.  **Verifica el Montaje**
    Confirma que el disco se ha montado correctamente y es accesible.

    ```bash
    df -h
    ```
    Deberías ver `/dev/sdb1` (o el nombre de tu partición) listado con su tamaño y uso, montado en `/mnt/backup_hdd`.

6.  **Configura el Montaje Automático (Persistencia)**
    Para asegurar que el Datastore se monte automáticamente después de cada reinicio, añade una entrada a `/etc/fstab`.

    Primero, obtén el UUID (Universally Unique Identifier) de la partición. Esto es más robusto que usar `/dev/sdX` ya que los nombres de los dispositivos pueden cambiar.

    ```bash
    blkid /dev/sdb1
    ```
    Copia el valor `UUID="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"`.

    Ahora, edita `/etc/fstab` con un editor de texto como `nano`:

    ```bash
    nano /etc/fstab
    ```
    Añade la siguiente línea al final del archivo, reemplazando el `UUID` con el valor que copiaste y ajustando la ruta y el tipo de sistema de archivos (`ext4` o `zfs` si lo configuraste así originalmente):

    ```
    UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX /mnt/backup_hdd ext4 defaults 0 2
    ```
    Guarda y cierra el archivo (Ctrl+O, Enter, Ctrl+X en `nano`).

    **Opcional: Prueba `fstab`**
    Puedes probar si la nueva entrada de `fstab` es correcta sin reiniciar:

    ```bash
    mount -a
    ```
    Si no hay errores, significa que la configuración es válida.

### Fase 2: Configuración del Datastore en la Nueva Instancia de PBS

Una vez que el disco externo está montado y accesible en el sistema de archivos del nuevo servidor PBS, el último paso es informarle a Proxmox Backup Server sobre este Datastore.

1.  **Accede a la Interfaz Web de PBS**
    Abre tu navegador web y navega a la dirección IP de tu **nueva** máquina PBS en el puerto 8007: `https://[IP_DEL_NUEVO_PBS]:8007`.

2.  **Inicia Sesión**
    Ingresa tus credenciales de usuario (ej., `root` y tu contraseña).

3.  **Navega a la Sección "Datastores"**
    En el menú de navegación de la izquierda, haz clic en **Datastores**.

4.  **Añade el Datastore Existente**
    Haz clic en el botón **Add Datastore**.
    *   **Name:** **¡MUY IMPORTANTE!** Debes introducir el **mismo nombre** que tenía tu Datastore en el servidor PBS original. Si no recuerdas el nombre, puedes listarlos haciendo `ls /mnt/backup_hdd/` en la terminal, y verás un directorio con el nombre del Datastore.
    *   **Path:** Introduce la ruta de montaje que configuraste en el paso 3 de la Fase 1, ej., `/mnt/backup_hdd`.
    *   **Owner:** Generalmente, `root@pam` (el valor predeterminado).
    *   **Comment (opcional):** Añade una descripción si lo deseas.

5.  **Confirma y Añade**
    Haz clic en el botón **Add**.

### Verificación Final

Una vez añadido, el nuevo servidor PBS escaneará automáticamente el Datastore. Deberías ver los mismos grupos de respaldo, snapshots y métricas que tenías en el servidor PBS original. Ahora puedes proceder a verificar la integridad de los respaldos y realizar restauraciones.

