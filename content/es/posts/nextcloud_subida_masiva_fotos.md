---
title: "NEXTCLOUD Importación Masiva"
date: 2025-12-11
draft: false
description: "Importación Masiva de Fotos a Nextcloud y Configuración de Memories"
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


# Guía Técnica: Importación Masiva de Fotos a Nextcloud y Configuración de Memories

**Objetivo:** Transferir una gran biblioteca de imágenes desde un almacenamiento externo a un servidor Nextcloud, asegurar su indexación correcta y visualizarlas cronológicamente en la aplicación **Memories**, evitando caídas del servidor por falta de recursos.

---

## 1. Transferencia de Archivos y Permisos

Lo primero es mover los datos físicos al directorio de datos de Nextcloud. No uses la interfaz web para cientos de gigabytes; usa la terminal.

### A. Montaje y Copia (Rsync)
Montamos el disco externo y copiamos los datos usando `rsync` para mantener la integridad y ver el progreso.

```bash
# 1. Montar disco externo (ejemplo)
sudo mount /dev/sdb1 /mnt/disco_externo

# 2. Copiar archivos a la carpeta del usuario de Nextcloud
# Ruta destino típica: /var/www/html/nextcloud/data/[USUARIO]/files/[CARPETA_DESTINO]
sudo rsync -avP /mnt/disco_externo/MisFotos/ /var/www/html/nextcloud/data/USER/files/Photos/
```

### B. Corrección de Permisos (Crítico)
Al copiar desde un externo, los archivos suelen pertenecer a `root`. Nextcloud (Apache/Nginx) necesita ser el dueño (`www-data`).

```bash
# Ajustar propietario recursivamente
sudo chown -R www-data:www-data /var/www/html/nextcloud/data/USER/files/Photos/

# Ajustar permisos de directorios (755) y archivos (644)
sudo find /var/www/html/nextcloud/data/USER/files/Photos/ -type d -exec chmod 755 {} \;
sudo find /var/www/html/nextcloud/data/USER/files/Photos/ -type f -exec chmod 644 {} \;
```

---

## 2. Optimización del Servidor (Prevención de Crashes)

El procesamiento de miles de imágenes (thumbnails, reconocimiento facial, transcodificación de video) consume mucha RAM. Sin esta configuración, MariaDB o PHP morirán por **OOM (Out Of Memory)**.

### A. Aumentar memoria SWAP
Si el servidor tiene <8GB de RAM, es obligatorio un archivo de intercambio (Swap) para absorber picos de carga.

```bash
# Crear swap de 2GB (o 4GB si hay mucho video)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
# Hacer persistente
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### B. Limitar la concurrencia en `config.php`
Editamos `/var/www/html/nextcloud/config/config.php` para evitar que Nextcloud intente procesar 20 fotos a la vez.

Añadir/Modificar al final del array `$CONFIG`:
```php
  'enable_previews' => true,
  'preview_max_x' => 2048,
  'preview_max_y' => 2048,
  'preview_concurrency_all' => 1, // CLAVE: Procesa 1 a 1 para no saturar CPU/RAM
  'preview_concurrency_new' => 1,
  'jpeg_quality' => 60,
```

---

## 3. Registro de Archivos en Nextcloud

Aunque los archivos están en el disco, la base de datos de Nextcloud no sabe que existen. Hay que escanearlos.

```bash
# Escanear archivos para el usuario específico (o --all para todos)
sudo -u www-data php /var/www/html/nextcloud/occ files:scan --all
```
*Salida esperada:*
> `+---------+-------+`
> `| Folders | Files |`
> `| 974     | 17101 |`

---

## 4. Configuración de la App Memories

Antes de indexar, debemos decirle a Memories qué carpetas debe incluir en la línea de tiempo.

1.  Ir a **Nextcloud Web** -> **Configuración de Administración** -> **Memories**.
2.  Buscar la sección **"Ruta de la Línea de tiempo"** (o General).
3.  **Seleccionar las carpetas**: Asegúrate de marcar la carpeta donde copiaste las fotos (ej: `/Photos` o `/_Contacts-Backup`).
    *   *Nota:* Si no se marcan aquí, las fotos existen en "Archivos" pero el Timeline de Memories saldrá vacío.

---

## 5. Indexación y Generación de Vistas Previas

El paso final es leer los metadatos EXIF (fechas de captura) y generar las miniaturas.

### A. Indexar Metadatos (Memories)
```bash
# -f fuerza el reescaneo, -vvv muestra detalles
sudo -u www-data php /var/www/html/nextcloud/occ memories:index -vvv
```

### B. Pre-generar Previews (Opcional pero recomendado)
Para que la navegación en el móvil sea fluida, generamos las miniaturas de antemano.
```bash
sudo -u www-data php /var/www/html/nextcloud/occ preview:generate-all -vvv
```

---

## Resolución de Problemas Comunes

**Error:** `SQLSTATE[HY000] [2002] Connection refused` durante el escaneo.
*   **Causa:** La base de datos se ha cerrado inesperadamente porque el servidor se quedó sin RAM (OOM Killer mató a MariaDB).
*   **Solución:** Reiniciar MariaDB (`sudo systemctl restart mariadb`), verificar que el SWAP está activo (`free -h`) y asegurar que `preview_concurrency` está en `1` en el config.php.

**Síntoma:** Las fotos aparecen en "Archivos" pero no en "Memories".
*   **Causa:** La carpeta no está seleccionada en la configuración de Memories.
*   **Solución:** Revisar el Paso 4 de esta guía.