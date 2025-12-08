---
title: "Backup en Proxmox"
date: 2024-11-10
draft: false
description: "Backup de una VM a HDD en Proxmox."
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
# Backup y Restauración de Máquinas Virtuales Proxmox VE (Guía Avanzada)

Este post detalla el proceso para realizar un backup completo de una Máquina Virtual (VM) en Proxmox VE a un disco externo o directorio no gestionado por Proxmox, y cómo restaurarla, manteniendo la distribución original de los discos en diferentes almacenamientos. Se asume un entorno Proxmox VE con ZFS pools (`local-zfs`, `dual-mirror-8_TB`) y un disco externo (`/dev/sda`) montado.

## Escenario de la VM a Respaldar (ID 100)

La VM con ID `100` (`NEXT`) tiene la siguiente configuración de discos:

```
scsi0: local-zfs:vm-100-disk-0,backup=1,iothread=1,size=50G
scsi1: dual-mirror-8_TB:vm-100-data-new,backup=0,discard=on,iothread=1,size=509G
```
Notamos que `scsi1` tiene `backup=0`, lo que significa que `vzdump` lo ignoraría por defecto.

## Preparación del Destino del Backup (`/dev/sda1`)

El disco de 3 TB (`/dev/sda`) se ha formateado con `ext4` y se ha montado en `/mnt/backup_3tb`.

```bash
# Verificación de la partición y punto de montaje
lsblk -f /dev/sda
df -h /mnt/backup_3tb

# Salida esperada (ejemplo):
# NAME        FSTYPE FSVER LABEL        UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
# sda
# └─sda1      ext4   1.0   Backup3TB    da4caf-da99-4886-8107-e5731d0090992 2.6T      1% /mnt/backup_3tb
```

## Proceso de Backup Completo de la VM 100

Dado que `vzdump` en algunas versiones de Proxmox puede no reconocer opciones avanzadas como `--all-volumes` o `--include-post-vmdisks` para discos con `backup=0`, la estrategia más robusta para un backup completo es modificar temporalmente la configuración de la VM.

**Pre-requisito:** La VM 100 debe estar **detenida**. (Verificar con `qm status 100`).

### Paso 1: Modificar Temporalmente la Configuración del Disco `scsi1`

Activamos la opción `backup=1` para el disco `scsi1` en la configuración de la VM 100. Esto asegura que `vzdump` lo incluya en el backup.

```bash
qm set 100 -scsi1 dual-mirror-8_TB:vm-100-data-new,backup=1,discard=on,iothread=1,size=509G
```

### Paso 2: Verificar la Configuración Modificada

Confirmamos que `scsi1` ahora tiene `backup=1`.

```bash
qm config 100
```
La salida debería mostrar: `scsi1: dual-mirror-8_TB:vm-100-data-new,backup=1,discard=on,iothread=1,size=509G`

### Paso 3: Ejecutar el Backup con `vzdump`

Utilizamos el comando `vzdump` con las opciones básicas y el destino directo para el archivo de backup.

```bash
vzdump 100 --mode stop --compress zstd --dumpdir /mnt/backup_3tb --tmpdir /var/tmp --maxfiles 3
```

*   `--mode stop`: La VM está detenida, garantizando un estado consistente del disco.
*   `--compress zstd`: Utiliza el algoritmo de compresión Zstandard para optimizar el tamaño del archivo.
*   `--dumpdir /mnt/backup_3tb`: Especifica el directorio de destino para el archivo de backup (el disco externo montado).
*   `--tmpdir /var/tmp`: Directorio temporal para `vzdump` (asegurar espacio suficiente).
*   `--maxfiles 3`: Mantiene las últimas 3 versiones del backup.

Durante la ejecución, la salida de `vzdump` confirmará la inclusión de ambos discos:

```
INFO: include disk 'scsi0' 'local-zfs:vm-100-disk-0' 50G
INFO: include disk 'scsi1' 'dual-mirror-8_TB:vm-100-data-new' 509G
# ... (progreso del backup) ...
INFO: backup is sparse: 357.83 GiB (64%) total zero data
INFO: transferred 559.00 GiB in 1345 seconds (425.6 MiB/s)
INFO: archive file size: 193.36GB
INFO: Backup job finished successfully
```

### Paso 4: Verificar el Archivo de Backup

Comprobamos que el archivo de backup se ha creado correctamente en el destino.

```bash
ls -lh /mnt/backup_3tb/
```
Se espera un archivo similar a `vzdump-qemu-100-YYYY_MM_DD-HH_MM_SS.vma.zst` con el tamaño comprimido (ej. `194G`).

### Paso 5: Revertir la Configuración del Disco `scsi1` (CRÍTICO)

Una vez completado el backup, es **imperativo** restaurar la configuración original del disco `scsi1` a `backup=0` para evitar comportamientos inesperados en futuros backups o operaciones de la VM.

```bash
qm set 100 -scsi1 dual-mirror-8_TB:vm-100-data-new,backup=0,discard=on,iothread=1,size=509G
```

### Paso 6: Verificar que la Configuración ha sido Revertida

```bash
qm config 100
```
La salida debería mostrar nuevamente: `scsi1: dual-mirror-8_TB:vm-100-data-new,backup=0,discard=on,iothread=1,size=509G`

---

## Proceso de Restauración del Backup (a VM ID 101)

Para probar la integridad del backup, restauraremos la VM en un nuevo ID (`101`), replicando la distribución original de los discos en sus respectivos almacenamientos.

### Paso 1: Identificar el Archivo de Backup y Nuevo ID

*   Archivo de backup: `/mnt/backup_3tb/vzdump-qemu-100-2025_11_10-15_28_24.vma.zst`
*   Nuevo ID de VM: `101` (Asegurarse de que no esté en uso con `qm status 101`).

### Paso 2: Ejecutar el Comando de Restauración `qmrestore`

Utilizamos la opción `--disk` para especificar dónde se restaurará cada disco del backup.

```bash
qmrestore /mnt/backup_3tb/vzdump-qemu-100-2025_11_10-15_28_24.vma.zst 101 \
  --disk scsi0:local-zfs \
  --disk scsi1:dual-mirror-8_TB
```

*   `--disk scsi0:local-zfs`: Restaura el disco `scsi0` al almacenamiento `local-zfs`.
*   `--disk scsi1:dual-mirror-8_TB`: Restaura el disco `scsi1` al almacenamiento `dual-mirror-8_TB`.

**Verificación de Espacio:** Asegúrese de que `local-zfs` tenga al menos 50GB libres y `dual-mirror-8_TB` al menos 509GB libres.

### Paso 3: Verificar la Configuración de la VM Restaurada

Una vez finalizada la restauración, se inspecciona la configuración de la VM 101.

```bash
qm config 101
```
La salida mostrará los discos restaurados en sus respectivos almacenamientos, por ejemplo:

```
# ...
scsi0: local-zfs:vm-101-disk-0,size=50G
scsi1: dual-mirror-8_TB:vm-101-disk-1,size=509G
# ...
```

### Paso 4: Arrancar y Validar la VM Restaurada

```bash
qm start 101
```
Acceda a la consola de la VM (vía la interfaz web de Proxmox o `qm console 101`) para confirmar que el sistema operativo arranca correctamente y que todos los datos en ambos discos son accesibles y consistentes.

### Paso 5: Limpieza (Post-Validación)

Una vez confirmada la funcionalidad del backup, se procede a eliminar la VM de prueba para liberar recursos.

```bash
qm stop 101      # Detener la VM si está en ejecución
qm destroy 101 --purge # Eliminar la VM y sus discos
```
La opción `--purge` es esencial para asegurar la eliminación de todos los volúmenes de disco asociados.

---

Este procedimiento garantiza un backup completo y una restauración fiable de VMs en Proxmox VE, incluso en escenarios con configuraciones de disco específicas y restricciones de versiones de herramientas.
```