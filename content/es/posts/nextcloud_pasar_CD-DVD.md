---
title: "NEXTCLOUD Importación Masiva CD y DVD"
date: 2025-12-18
draft: false
description: "Importación Masiva de Fotos a Nextcloud CD y DVD"
hideToc: false
enableToc: false
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags: 
- Proxmox
- nextcloud
- debian
- linux
- homelab
image: images/feature1/nextcloud_logo.png
---
![Nextcloud custom login server](/images/post/nextcloud_custom.png)

---

# Rescatando recuerdos digitales: Migración masiva de CDs antiguos a Nextcloud

Recientemente me embarqué en el proyecto de digitalizar y centralizar años de fotografías (muchas en formato RAW .NEF) almacenadas en CDs antiguos hacia un servidor casero con **Nextcloud** y la aplicación **Memories**.


## 1. El escenario base: Montaje y copia simple

Para los discos en buen estado, el proceso en Linux (Debian/Ubuntu) es directo. Identificamos la unidad y montamos.

```bash
# Identificar la unidad (usualmente sr0)
lsblk

# Crear punto de montaje y montar
mkdir -p /media/cdrom
mount /dev/sr0 /media/cdrom
```

Para la transferencia, `rsync` es superior a `cp` porque permite verificar tamaños y reanudar si algo falla:

```bash
# Copia recursiva verificando por tamaño
rsync -rv --size-only /media/cdrom/ /mnt/nextcloud_data/usuario/files/Photos/
```

Una vez copiados, Nextcloud no ve los archivos mágicamente. Hay que indexarlos manualmente:

```bash
# Escaneo de archivos (Ajustar ruta y usuario según instalación)
sudo -u www-data php /var/www/html/nextcloud/occ files:scan --all
```

## 2. Obstáculo: Incompatibilidad de versiones PHP

Mi sistema tenía instalada una versión muy reciente de PHP (8.5) por defecto en el CLI, pero Nextcloud requiere una versión estable (ej. 8.3). Esto causaba errores al ejecutar comandos `occ`.

**Solución:** Invocar explícitamente el binario de la versión correcta.

```bash
# En lugar de 'php', usamos la ruta completa
sudo -u www-data /usr/bin/php8.3 /var/www/html/nextcloud/occ files:scan --all
```

También fue necesario ajustar el **Crontab** para evitar que las tareas de fondo fallaran silenciosamente:

```bash
crontab -u www-data -e
```
*Contenido corregido:*
```bash
*/5 * * * * /usr/bin/php8.3 -f /var/www/html/nextcloud/cron.php
*/10 * * * * /usr/bin/php8.3 /var/www/html/nextcloud/occ preview:pre-generate
```

## 3. Visualización de archivos RAW (.NEF)

Al subir archivos Nikon RAW (.NEF), el navegador no podía mostrarlos. Fue necesario activar **ImageMagick** y configurar Nextcloud.

1.  **Instalación:** `apt install imagemagick php8.3-imagick`
2.  **Configuración:** Editar `/var/www/html/nextcloud/config/config.php`.



```php
<?php
$CONFIG = array (
  // ... otras configuraciones ...
  'enable_previews' => true,
  'enabledPreviewProviders' =>
  array (
    0 => 'OC\\Preview\\HEIC',
    1 => 'OC\\Preview\\Image',
    2 => 'OC\\Preview\\TIFF',
    3 => 'OC\\Preview\\Movie',
    4 => 'OC\\Preview\\Imagick', // <--- Clave para RAW/NEF
  ),
  // ...
);
```

3.  **Generación masiva:**
    ```bash
    sudo -u www-data /usr/bin/php8.3 /var/www/html/nextcloud/occ preview:generate-all -vvv
    ```

## 4. El desafío final: CDs dañados (Input/Output Error)

Varios discos presentaban errores de lectura (`Input/output error (5)` en rsync). Aunque intenté crear imágenes ISO con `ddrescue`, el sistema de archivos lógico estaba corrupto, haciendo imposible montar o copiar los archivos de forma tradicional.

**Solución definitiva: PhotoRec**
La herramienta **PhotoRec** (del paquete `testdisk`) ignora el sistema de archivos dañado y lee los sectores crudos buscando cabeceras de archivos (JPG, NEF, etc.).

### Flujo de trabajo para recuperación masiva

Para procesar varios discos dañados en serie, diseñé este procedimiento:

1.  **Preparación:**
    ```bash
    mkdir -p /root/recuperacion_cd
    ```

2.  **Extracción (Por cada disco):**
    *   Insertar disco y esperar a que cargue.
    *   Limpiar temporales anteriores: `rm -rf /root/recuperacion_cd/recup_dir*`
    *   Ejecutar: `photorec /dev/sr0`
    *   *Opciones:* Seleccionar unidad -> Partition "Unknown/Whole Disk" -> Filesystem "Other" -> Destino `/root/recuperacion_cd`.

3.  **Organización y Mover:**
    Como PhotoRec recupera archivos con nombres genéricos (`f001.jpg`), es vital moverlos a carpetas separadas en Nextcloud inmediatamente.

    ```bash
    # Crear carpeta destino específica para este CD
    mkdir -p /mnt/nextcloud_data/usuario/files/Photos/CD_Recuperado_01

    # Mover archivos
    mv /root/recuperacion_cd/recup_dir.*/* /mnt/nextcloud_data/usuario/files/Photos/CD_Recuperado_01/
    ```

4.  **Finalización:**
    Ajustar permisos y notificar a Nextcloud.

    ```bash
    chown -R www-data:www-data /mnt/nextcloud_data/usuario/files/Photos/
    sudo -u www-data /usr/bin/php8.3 /var/www/html/nextcloud/occ files:scan --all
    sudo -u www-data /usr/bin/php8.3 /var/www/html/nextcloud/occ memories:index
    ```

## Conclusión

Migrar medios físicos antiguos no es solo "copiar y pegar". Requiere:
*   Consistencia en las versiones de software (PHP).
*   Configuración específica para formatos profesionales (RAW).
*   Herramientas forenses (PhotoRec) cuando el hardware falla.

