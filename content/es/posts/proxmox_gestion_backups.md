---
title: "Gestión de backups "
date: 2025-06-15
draft: false
description: "Gestión de backups desde la shell"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
  - homelab
  - privacidad
  - selfhosting
  - opensource
  - zfs
  - raid
categories:
  - Tecnología
series:
  - Homelab desde cero
libraries:
  - mermaid
image: images/feature2/proxmox2.png
---
intro

## Eliminar todos los backups de un ID LXC

### **Importante:** ¿Dónde guardas tus backups?

Este método funciona para almacenamientos basados en archivos, como **`Directory`**, **`NFS`** o **`CIFS`**.

**¡Atención!** Si estás usando **Proxmox Backup Server (PBS)**, este método **NO** funcionará y no debes intentarlo. Los backups en PBS se gestionan a través de su propia interfaz o con el comando `proxmox-backup-client`. El método que te describo a continuación es para los backups tradicionales de Proxmox (archivos `.vma.zst`, `.tar.gz`, etc.).

---

### Paso 1: Conéctate al Shell de tu Host Proxmox

Puedes hacerlo a través de la interfaz web (`TuNodo` -> `Shell`) o conectándote directamente por SSH.

### Paso 2: Identifica la Ruta Exacta de tu Almacenamiento de Backups

Los archivos de backup se guardan en un subdirectorio llamado `dump` dentro de la ruta que configuraste para tu almacenamiento.

Para encontrar la ruta:
1.  Ve a la interfaz web de Proxmox: `Datacenter` -> `Storage`.
2.  Busca tu almacenamiento de backups en la lista.
3.  Fíjate en la columna **`Path`**. Esa es la ruta base.

**Ejemplo:**
Si tu almacenamiento se llama `backups_locales` y su `Path` es `/mnt/backups`, entonces los archivos de backup estarán en:
`/mnt/backups/dump/`

Si usas el almacenamiento `local` por defecto, la ruta suele ser:
`/var/lib/vz/dump/`

Para los siguientes pasos, usaré `/ruta/de/tus/backups/` como ejemplo. 

### Paso 3: El Paso de Verificación (¡El más importante!)

**Nunca borres nada sin antes verificar qué vas a borrar.** Usaremos el comando `ls` para listar todos los archivos que coinciden con el patrón de backups del contenedor 101.

Los archivos de backup de Proxmox siguen un patrón: `vzdump-lxc-ID-FECHA-HORA.extension`

Ejecuta el siguiente comando (reemplazando la ruta):

```bash
ls -lh /ruta/de/tus/backups/dump/vzdump-lxc-101-*
```

**Desglose del comando:**
*   `ls`: Comando para listar archivos.
*   `-lh`: Opciones para mostrar el tamaño en formato legible (`-h`) y en una lista detallada (`-l`).
*   `/ruta/de/tus/backups/dump/`: La ruta a la carpeta de backups. **No olvides el `/dump/` al final.**
*   `vzdump-lxc-101-*`: Este es el patrón mágico.
    *   `vzdump-lxc-`: Prefijo para backups de contenedores LXC.
    *   `101`: El ID de tu contenedor.
    *   `-`: Separador.
    *   `*`: El comodín. Le dice a la shell "coincide con cualquier cosa que venga después". Esto incluirá las fechas, horas y extensiones (`.tar.zst`, etc.) de todos los backups del CT 101.

**Revisa la salida de este comando.** Deberías ver una lista de todos y cada uno de los archivos de backup del contenedor 101, y ningún otro. Si la lista es correcta, puedes proceder al siguiente paso.

### Paso 4: La Eliminación Definitiva

Una vez que has verificado con `ls` que el patrón es correcto, simplemente reemplaza `ls -lh` por `rm -v`.

```bash
rm -v /ruta/de/tus/backups/dump/vzdump-lxc-101-*
```

**Desglose del comando:**
*   `rm`: El comando para eliminar (remover) archivos. **¡Este comando es irreversible!**
*   `-v`: Opción de "verbose" (detallado). Imprimirá en pantalla el nombre de cada archivo a medida que lo borra. Esto te da una confirmación visual de lo que se está haciendo.

La terminal te mostrará algo como:
```
removed '/ruta/de/tus/backups/dump/vzdump-lxc-101-2024_01_10-10_00_01.tar.zst'
removed '/ruta/de/tus/backups/dump/vzdump-lxc-101-2024_01_11-10_00_02.tar.zst'
removed '/ruta/de/tus/backups/dump/vzdump-lxc-101-2024_01_12-10_00_01.tar.zst'
```

## Eliminar todos los backups de un ID VM

El proceso para las máquinas virtuales (VMs) es muy similar al de los contenedores LXC. La herramienta principal y la lógica son las mismas, solo cambia un pequeño detalle en el nombre de los archivos.

Usaremos la herramienta `pvesm` (Proxmox VE Storage Manager), que es la forma correcta y segura de gestionar volúmenes (como los backups) desde la shell.

### Resumen Rápido

La diferencia principal entre un backup de LXC y uno de VM es el nombre del archivo:

*   **LXC Container:** `vzdump-lxc-CONTAINER_ID-...`
*   **VM (QEMU):** `vzdump-qemu-VM_ID-...`

### Instrucciones Paso a Paso para Eliminar Backups de VMs

#### Paso 1: Identificar el ID del Storage

Primero, necesitas saber el nombre (ID) del almacenamiento donde se guardan tus backups. Puedes verlo en la interfaz web de Proxmox en `Datacenter -> Storage` o con este comando en la shell:

```bash
pvesm status
```

El resultado será algo así. Fíjate en la columna `Storage`:

```
Name              Type     Status           Total            Used       Available        %
local             dir      active      98226068        31046484      62143004   31.61%
local-backups     dir      active     960348088       219277564     692254540   22.83%
```

En este ejemplo, mis backups están en `local-backups`.

#### Paso 2: Listar los Backups de una VM Específica

Ahora, vamos a listar todos los backups que existen para una máquina virtual concreta. Necesitas el **ID del Storage** (del paso 1) y el **ID de la VM**.

Usa el siguiente comando, reemplazando `<STORAGE_ID>` y `<VM_ID>`:

```bash
pvesm list <STORAGE_ID> --vmid <VM_ID>
```

**Ejemplo práctico:** Para listar los backups de la VM con ID `101` que están en el storage `local-backups`:

```bash
pvesm list local-backups --vmid 101
```

La salida se verá así, mostrando cada backup como un "VolId":

```
VolId                                           Format      Type         Size  VMID
local-backups:backup/vzdump-qemu-101-2025_06_14-11_30_01.vma.zst    vma  backup    17179869184   101
local-backups:backup/vzdump-qemu-101-2025_06_15-11_30_02.vma.zst    vma  backup    17205211136   101
local-backups:backup/vzdump-qemu-101-2025_06_16-11_30_03.vma.zst    vma  backup    17231265792   101
```
La columna `VolId` contiene el identificador único que necesitas para el siguiente paso.

#### Paso 3: Eliminar el Backup Seleccionado

Este es el comando de eliminación. Usa el comando `pvesm free`. Es crucial que copies el `VolId` **exacto** del listado anterior.

La sintaxis es:

```bash
pvesm free <VolId>
```

**Ejemplo práctico:** Para eliminar el backup más antiguo del ejemplo anterior (`2025_06_14`):

```bash
pvesm free local-backups:backup/vzdump-qemu-101-2025_06_14-11_30_01.vma.zst
```

Proxmox te pedirá confirmación (a menos que uses la opción `--force`, no recomendada a menos que estés en un script). Tras confirmar, el backup se eliminará del disco y del índice de Proxmox.

### Poniéndolo todo junto: Un caso de uso completo

**Objetivo:** Eliminar el backup más antiguo de la VM 105, que está en el storage `nfs-storage`.

1.  **Listar los backups:**
    ```bash
    pvesm list nfs-storage --vmid 105
    ```
    *Salida de ejemplo:*
    ```
    VolId                                                       Format  Type        Size  VMID
    nfs-storage:backup/vzdump-qemu-105-2025_05_01-22_00_01.vma.zst   vma  backup   8589934592   105
    nfs-storage:backup/vzdump-qemu-105-2025_06_01-22_00_01.vma.zst   vma  backup   8623489024   105
    ```

2.  **Identificar el `VolId` a eliminar:** `nfs-storage:backup/vzdump-qemu-105-2025_05_01-22_00_01.vma.zst`

3.  **Ejecutar el comando de eliminación:**
    ```bash
    pvesm free nfs-storage:backup/vzdump-qemu-105-2025_05_01-22_00_01.vma.zst
    ```

### Método Alternativo (No Recomendado)

Al igual que con los LXC, podrías ir directamente al directorio de backups y usar `rm` para borrar el archivo.

1.  Encuentra la ruta del storage con `pvesm status`.
2.  Navega a esa ruta. Por defecto, para el storage `local` suele ser `/var/lib/vz/dump/`.
3.  Usa `rm` para borrar el archivo:
    ```bash
    # ¡CUIDADO! Este comando es destructivo e irreversible
    rm /var/lib/vz/dump/vzdump-qemu-101-....vma.zst
    ```

**¿Por qué no se recomienda?** Porque esto solo borra el archivo del disco. El backup podría seguir apareciendo en la interfaz gráfica de Proxmox hasta que el almacenamiento se vuelva a escanear (o "prune"). Usar `pvesm free` es más limpio porque actualiza el índice de Proxmox al instante.

### Puntos Clave y Precauciones

*   **¡La eliminación es permanente!** No hay papelera de reciclaje.
*   Siempre usa `pvesm list` primero para asegurarte de que tienes el nombre de archivo (`VolId`) correcto.
*   Copiar y pegar el `VolId` es la forma más segura de evitar errores de tipeo.
*   Recuerda la diferencia: `qemu` en el nombre de archivo es para VMs, `lxc` es para contenedores.