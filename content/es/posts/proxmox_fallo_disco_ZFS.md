---
title: "Disco ZFS Fallando: Detección y Reemplazo"
date: 2025-06-15
draft: false
description: "Tutorial paso a paso sobre cómo detectar, identificar físicamente y reemplazar un disco duro con fallos en un pool ZFS"
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
image: images/feature2/nascrash.png
---

 ZFS, es un sistema de archivos robusto diseñado precisamente para estas situaciones. Recientemente, mi servidor comenzó a reportar errores en uno de los discos de mi pool.

Compartiré el proceso exacto que seguí para detectar el problema, identificar el disco físico culpable y reemplazarlo de forma segura, todo sin perder un solo byte de datos.

## Paso 1: Detección del Problema con `zpool status`

El primer indicio de un problema suele venir del comando `zpool status`. Al ejecutarlo, recibí una salida que, aunque preocupante, me dio toda la información necesaria para empezar.

```bash
usuario@servidor:~# zpool status mi-pool-raid10
  pool: mi-pool-raid10
 state: ONLINE
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
        using 'zpool clear' or replace the device with 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-9P
  scan: scrub repaired 56.2M in 00:14:03 with 0 errors on [Fecha del escaneo]
config:

        NAME              STATE     READ WRITE CKSUM
        mi-pool-raid10    ONLINE       0     0     0
          mirror-0        ONLINE       0     0     0
            sda           ONLINE       0     0     0
            sdb           ONLINE       0     0     0
          mirror-1        ONLINE       0     0     0
            sdc           ONLINE       0     0     0
            sdd           ONLINE       0     0 3.53K

errors: No known data errors
```



**Análisis de la salida:**

*   **`state: ONLINE`**: ¡Buenas noticias! El pool sigue en línea y funcionando.
*   **`status: One or more devices has experienced an unrecoverable error`**: Aquí está la alerta. Algo no va bien.
*   **`sdd ONLINE 0 0 3.53K`**: El culpable. El disco `/dev/sdd` tiene más de 3.500 errores de checksum (CKSUM). Esto significa que ZFS intentó leer datos de él, pero no coincidían con la suma de verificación guardada. Gracias a la redundancia del `mirror-1`, pudo corregir el error al vuelo usando los datos del disco `sdc`.
*   **`errors: No known data errors`**: Lo más importante. Mis datos están íntegros.

Intenté ejecutar `zpool clear mi-pool-raid10`, pero los errores volvieron a aparecer, una señal inequívoca de que el disco tenía un fallo de hardware persistente. Era hora de reemplazarlo.
Otro ejemplo de errores encontrados,
```bash
usuario@server:~# zpool status zfs-raid10
  pool: zfs-raid10
 state: DEGRADED
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
        using 'zpool clear' or replace the device with 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-9P
  scan: resilvered 155M in 00:00:03 with 0 errors on [Fecha del escaneo]
config:

        NAME                                      STATE     READ WRITE CKSUM
        zfs-raid10                            DEGRADED     0     0     0
          mirror-0                            DEGRADED     0     0     0
            sda                                  DEGRADED     0     0     0  too many slow I/Os
            sdb                                 ONLINE           0     0     0
          mirror-1                            DEGRADED     0     0     0
            sdc                                  DEGRADED     0     0     1  too many errors
            ata-TOSHIBA                   ONLINE          0     0     0

errors: No known data errors
usuario@server:~# zpool clear zfs-raid10
usuario@server:~# 
usuario@server:~# 
usuario@server:~# zpool status zfs-raid10
  pool: zfs-raid10
 state: ONLINE
  scan: resilvered 155M in 00:00:03 with 0 errors on [Fecha del escaneo]
config:

        NAME                                  STATE     READ WRITE CKSUM
        zfs-raid10                        ONLINE       0     0     0
          mirror-0                        ONLINE       0     0     0
            sda                              ONLINE       0     0     0
            sdb                             ONLINE       0     0     0
          mirror-1                        ONLINE       0     0     0
            sdc                              ONLINE       0     0     0
            ata-TOSHIBA               ONLINE       0     0     0

errors: No known data errors
usuario@server:~# 
```
Otro fallo de disco
```bash
usuario@servidor:~# zpool status zfs-raid10
  pool: zfs-raid10
 state: DEGRADED
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
        using 'zpool clear' or replace the device with 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-9P
  scan: resilvered 155M in 00:00:03 with 0 errors on Mon Sep  8 20:38:48 2025
config:

        NAME                                  STATE     READ WRITE CKSUM
        zfs-raid10                        DEGRADED     0     0     0
          mirror-0                            DEGRADED     0     0     0
            sda                               DEGRADED     0     0    35  too many errors   <-- OJO
            sdb                               ONLINE       0     0     0
          mirror-1                            ONLINE       0     0     0
            sdc                               ONLINE       0     0    24
            ata-TOSHIBA_              ONLINE       0     0     0

errors: No known data errors
usuario@servidor:~# ^C
usuario@servidor:~# zpool clear
missing pool name
usage:
        clear [[--power]|[-nF]] <pool> [device]
usuario@servidor:~# zpool clear 
rpool           zfs-raid10  
usuario@servidor:~# zpool clear zfs-raid10 
usuario@servidor:~# 
usuario@servidor:~# 
usuario@servidor:~# zpool status zfs-raid10
  pool: zfs-raid10
 state: ONLINE
  scan: resilvered 155M in 00:00:03 with 0 errors on Mon Sep  8 20:38:48 2025
config:

        NAME                                  STATE     READ WRITE CKSUM
        zfs-raid10                        ONLINE       0     0     0
          mirror-0                            ONLINE       0     0     0
            sda                               ONLINE       0     0     0
            sdb                               ONLINE       0     0     0
          mirror-1                            ONLINE       0     0     0
            sdc                               ONLINE       0     0     0
            ata-TOSHIBA_              ONLINE       0     0     0

errors: No known data errors
usuario@servidor:~# 
```

## Paso 2: Identificación Física del Disco

Saber que `/dev/sdd` es el disco problemático no me ayuda a encontrarlo dentro de una caja con varios discos idénticos. Necesitaba un identificador único: el **número de serie**.

### Método A: El Número de Serie (El más fiable)

La herramienta `smartctl` es perfecta para esto.

```bash
usuario@servidor:~# smartctl -i /dev/sdd
smartctl 7.3 2022-02-28 r5338 [x86_64-linux-X.X.X] (local build)
Copyright (C) 2002-22, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Seagate Enterprise Series
Device Model:     ST4000NMXXXX-XXXXXX
Serial Number:    ABC123XYZ
LU WWN Device Id: 5 000c50 0XXXXXXXX
Firmware Version: EN09
User Capacity:    4,000,787,030,016 bytes [4.00 TB]
# ... resto de la salida
```

Con el número de serie **`ABC123XYZ`** en mano, solo tuve que buscar la etiqueta en cada uno de los discos físicos de mi servidor hasta encontrar el correcto.

A las 48 horas el error era ya desastroso.

```bash
usuario@servidor:~# smartctl -i /dev/sdd
smartctl 7.3 2022-02-28 r5338 [x86_64-linux-6.8.12-11-pve] (local build)
Copyright (C) 2002-22, Bruce Allen, Christian Franke, www.smartmontools.org

Short INQUIRY response, skip product id
A mandatory SMART command failed: exiting. To continue, add one or more '-T permissive' options.
usuario@servidor:~#
```
Esto pasa por comprar discos recuperados.

### Otros Métodos de Identificación, Si tienes LED en tu NAS.

Si el número de serie no fuera accesible, existen otras técnicas:

1.  **Hacer parpadear el LED del disco:** En hardware de servidor, el comando `ledctl locate=/dev/sdd` puede encender un LED de localización en la bahía del disco.
2.  **Generar actividad en el disco:** Con el comando `sudo dd if=/dev/sdd of=/dev/null bs=1M` podemos forzar una lectura constante. El LED de actividad del disco culpable parpadeará frenéticamente, delatando su posición. **¡Mucho cuidado al escribir este comando!**

## Paso 3: El Proceso de Reemplazo

Con el disco identificado, el proceso de reemplazo es metódico y seguro.

Mi estructura de RAID-10 se ve así:

{{< mermaid >}}
graph TD
    Pool[mi-pool-raid10] --> Mirror0[Mirror-0]
    Pool --> Mirror1[Mirror-1]
    Mirror0 --> sda[sda: Disco 1]
    Mirror0 --> sdb[sdb: Disco 2]
    Mirror1 --> sdc[sdc: Disco 3]
    Mirror1 --> sdd((sdd: Disco 4 - CON FALLOS))
{{< /mermaid >}}

1.  **Adquirir un disco de reemplazo:** Compré un disco nuevo de capacidad **igual o superior** (4 TB en mi caso). Es un requisito indispensable para ZFS.

2.  **Reemplazo Físico:** Apagué el servidor, localicé el disco con el número de serie `ABC123XYZ`, lo desconecté y lo sustituí por el nuevo en la **misma bahía y con los mismos cables**.

3.  **Reemplazo Lógico en ZFS:** Una vez encendido el servidor, el nuevo disco fue detectado como `/dev/sdd`. El siguiente comando le ordena a ZFS que reemplace el dispositivo "viejo" (identificado por su path) con el nuevo que ahora está en su lugar.

    ```bash
    sudo zpool replace mi-pool-raid10 sdd
    ```
    Si el nuevo disco apareciera con otro nombre (ej. `/dev/sdh`), se usaría el ID del disco viejo: `sudo zpool replace mi-pool-raid10 [ID-del-disco-viejo] /dev/sdh`.

4.  **Monitorizar la Reconstrucción (Resilvering):** ZFS inició inmediatamente el proceso de "resilvering", que consiste en copiar los datos del disco sano del espejo (`sdc`) al nuevo disco. Se puede monitorizar el progreso en tiempo real.

    ```bash
    watch zpool status
    ```
    La salida mostrará el porcentaje de reconstrucción y el tiempo estimado. Para un disco de 4 TB, este proceso puede tardar varias horas.

Una vez finalizado el `resilver`, el pool volvió a su estado `ONLINE` y saludable, sin errores y con todos los datos intactos.



---

### **Guía Técnica: Reemplazo de un Disco Fallido en un Pool ZFS RAID10**

#### **Introducción**

Este documento detalla el procedimiento completo seguido para reemplazar un disco duro fallido en un pool ZFS configurado en RAID10. El proceso abarca desde la detección inicial del fallo hasta la verificación final del estado del pool, incluyendo los imprevistos encontrados y las lecciones aprendidas. El objetivo es servir como guía de mejores prácticas para administradores de sistemas que enfrenten una situación similar.

---

#### **1. Detección y Diagnóstico Inicial del Fallo**

La monitorización proactiva del estado del pool ZFS es fundamental. El primer indicio del problema se obtuvo mediante el comando estándar de estado:

```bash
# zpool status mi-pool-raid10
```

La salida fue inequívoca, indicando que uno de los dispositivos estaba en estado `FAULTED` debido a un número excesivo de errores de lectura/escritura y checksum.

```
  pool: mi-pool-raid10
 state: ONLINE
status: One or more devices are faulted in response to persistent errors.
...
config:
  NAME          STATE     READ WRITE CKSUM
  mi-pool-raid10  ONLINE       0     0     0
    mirror-0    ONLINE       0     0     0
      disco_sano_1 ONLINE       0     0     0
      disco_sano_2 ONLINE       0     0     0
    mirror-1    ONLINE       0     0     0
      disco_sano_3 ONLINE       0     0     0
      disco_malo  FAULTED      3    80 3.53K  too many errors
```

El pool permanecía `ONLINE` gracias a la redundancia del `mirror-1`, pero en un estado degradado que requería acción inmediata.

---

#### **2. Identificación Física del Disco Defectuoso**

Un identificador lógico como `/dev/sdX` es efímero y no fiable para la identificación física, ya que puede cambiar en cada arranque. Es imperativo utilizar un identificador persistente.

**Precaución:** Nunca se debe asumir que `/dev/sdX` corresponde a una bahía física específica sin una verificación rigurosa.

Se emplearon dos métodos para la identificación inequívoca:

1.  **Comando `ls -l /dev/disk/by-id/`:** Este comando lista los enlaces simbólicos persistentes a los dispositivos, incluyendo el modelo y número de serie en el nombre del enlace. Se buscó el enlace que apuntaba al dispositivo lógico `disco_malo`.

    ```bash
    # ls -l /dev/disk/by-id/
    ...
    lrwxrwxrwx 1 root root 9 Jun 17 12:00 ata-SEAGATE_MODEL_SERIALNUMBER -> ../../disco_malo
    ...
    ```

2.  **Comando `smartctl -i /dev/disco_malo`:** Se utilizó para obtener directamente la información de identidad del disco, incluyendo el número de serie. El disco fallido estaba tan degradado que este comando falló, confirmando la severidad del problema y reforzando la identificación obtenida con el método anterior.

Con el número de serie en mano, se procedió a la localización física del disco en el chasis del servidor.

---

#### **3. Procedimiento de Reemplazo Físico**

Se siguió un protocolo de reemplazo "cold-swap" para minimizar riesgos:

1.  **Apagado Programado:** Se realizó un apagado controlado del servidor.
2.  **Extracción:** Se retiró el disco físicamente identificado por su número de serie.
3.  **Instalación:** Se instaló un nuevo disco de capacidad igual o superior en la misma bahía, utilizando los mismos cables de datos y alimentación.
4.  **Encendido:** Se arrancó el servidor.

---

#### **4. Incidente Crítico: Detección de Doble Fallo**

Tras el reinicio, una revisión del estado del pool reveló una situación crítica:

```
# zpool status mi-pool-raid10
...
status: One or more devices has experienced an error resulting in data corruption.
...
  mirror-1                DEGRADED
    disco_fantasma        FAULTED      ... was /dev/sdX
    disco_que_era_sano    ONLINE       ... 28.2K (CKSUM errors)
```

El disco que formaba el espejo del dispositivo recién fallado (`disco_que_era_sano`) comenzó a registrar miles de errores de checksum (`CKSUM`). Esto indicaba un **fallo casi simultáneo del segundo miembro del espejo**, la peor contingencia posible en un RAID1.

**Lección Aprendida:** Un fallo en un disco de un mirror ejerce una carga adicional sobre el disco restante durante la degradación y el `resilvering`. Si este segundo disco ya estaba envejecido o tenía problemas latentes, el riesgo de un fallo en cascada es significativo.

---

#### **5. Recuperación y Reconstrucción del Pool**

La estrategia se centró en estabilizar el pool y reemplazar el disco fallido original, confiando en que el segundo disco aguantaría el proceso de reconstrucción.

1.  **Revisión de Conexiones:** Se realizó un nuevo ciclo de apagado para revisar y reasentar firmemente las conexiones de todos los discos, lo cual mejoró notablemente el estado del segundo disco afectado.
2.  **Comando de Reemplazo:** Se ejecutó el comando `zpool replace`, utilizando el identificador numérico del disco ausente y la ruta persistente (`by-id`) del nuevo disco para evitar ambigüedades.

    ```bash
    # zpool replace mi-pool-raid10 <GUID_DISCO_AUSENTE> /dev/disk/by-id/<ID_DISCO_NUEVO>
    ```

3.  **Monitorización del `Resilvering`:** El proceso de reconstrucción se inició inmediatamente. Se monitorizó en tiempo real con `watch zpool status`.

    ```
    scan: resilver in progress... XX% done, HH:MM:SS to go
    ```

Al finalizar, el pool volvió al estado `ONLINE`, aunque con contadores de error persistentes del incidente.

---

#### **6. Limpieza Final y Verificación de Integridad**

Para devolver el pool a un estado impecable, se siguieron dos pasos finales:

1.  **Auditoría de Datos (`scrub`):** Se inició un `scrub` manual para forzar a ZFS a leer cada bloque de datos y verificar su integridad contra la copia redundante.

    ```bash
    # zpool scrub mi-pool-raid10
    ```

2.  **Limpieza de Errores (`clear`):** Una vez verificado que el `scrub` no reportaba errores incorregibles, se reiniciaron los contadores de errores del pool. Esto es crucial para que cualquier futuro error sea indicativo de un nuevo problema.

    ```bash
    # zpool clear mi-pool-raid10
    ```

Tras estos pasos, el comando `zpool status` finalmente reportó un estado `ONLINE` limpio, sin errores pendientes y con todos los discos funcionando correctamente.

---

#### **7. Post-Recuperación: Estandarización de Identificadores**

Para mejorar la robustez y legibilidad de la configuración, se procedió a actualizar las referencias de los discos antiguos (que usaban `/dev/sdX`) a sus rutas persistentes `/dev/disk/by-id/`.

**Procedimiento:** Se realizó un "auto-reemplazo" para cada disco, **de uno en uno**, esperando a que cada `resilver` terminara antes de proceder con el siguiente.

```bash
# zpool replace -f mi-pool-raid10 /dev/sda /dev/disk/by-id/ata-SEAGATE_MODEL_SERIAL_SDA
```
**Nota:** El flag `-f` (force) es necesario para que ZFS acepte reemplazar un disco por sí mismo.

Al finalizar este proceso, la configuración del pool muestra exclusivamente identificadores persistentes, haciéndolo inmune a cambios de nombre de dispositivo tras reinicios.

#### **Conclusión**

El reemplazo de un disco en un pool ZFS, aunque teóricamente sencillo, puede presentar complicaciones graves. Esta experiencia subraya la importancia de:
*   **Monitorización constante.**
*   **Identificación física precisa mediante números de serie.**
*   **Uso exclusivo de identificadores persistentes (`/dev/disk/by-id/`) en las operaciones de ZFS.**
*   **Actuar con rapidez ante el primer fallo**, ya que el riesgo de fallos en cascada es real.
*   **Mantener copias de seguridad externas y verificadas**, ya que la redundancia no es infalible.



**Reemplazo del Disco `/dev/sda`**

1.  **Identifica el disco físico `sda`:**
    *   Usa `ls -l /dev/disk/by-id/` para ver los identificadores persistentes de tus discos y encontrar cuál apunta a `/dev/sda`. Esto te ayudará a identificar el disco físico correcto en tu servidor.
    *   `ls -l /dev/disk/by-id/ | grep sda`
    *   Una vez que lo tengas, busca el disco físico correspondiente.

2.  **Prepara el nuevo disco:**
    *   Asegúrate de que el nuevo disco no contenga datos importantes ni particiones que quieras conservar.

3.  **Inicia el reemplazo del disco en ZFS:**
    *   Ejecuta el siguiente comando para decirle a ZFS que reemplace `sda` con el nuevo disco. **Asegúrate de que `/dev/sdX_NUEVO` sea el identificador correcto del nuevo disco (por ejemplo, `/dev/sde`, `/dev/sdf`, etc.).** Si tienes un identificador `/dev/disk/by-id/` para el nuevo disco, es más seguro usarlo.

    ```bash
    sudo zpool replace zfs-raid10-8tb sda /dev/sdX_NUEVO
    ```

    *   **Ejemplo:** Si el nuevo disco se detecta como `/dev/sde`, el comando sería:
        ```bash
        sudo zpool replace zfs-raid10-8tb sda /dev/sde
        ```

4.  **Monitorea el proceso de "resilver":**
    *   Después de ejecutar el comando `zpool replace`, ZFS comenzará a copiar todos los datos del otro disco del espejo (`sdb`) al nuevo disco (`/dev/sdX_NUEVO`). Este proceso se llama "resilver".
    *   Puedes monitorear el progreso con:
        ```bash
        zpool status zfs-raid10-8tb
        ```
    *   Verás que el pool estará en estado `DEGRADED` durante el resilver (ya que un disco está siendo reemplazado), y el nuevo disco aparecerá como "replacing". Una vez completado, el pool volverá a `ONLINE`.

5.  **Limpia los errores (opcional pero recomendado):**
    *   Una vez que el "resilver" haya terminado y el pool vuelva a estar `ONLINE` (sin dispositivos en estado `DEGRADED` o "replacing"), puedes limpiar los errores registrados:
        ```bash
        sudo zpool clear zfs-raid10-8tb
        ```
    *   Esto restablecerá el contador de errores del pool.

**Nota importante:** Aunque solo estés reemplazando `sda` ahora, recuerda que el disco `sdd` tiene 21 sectores reasignados, lo cual es una señal clara de falla. Es muy recomendable que también planifiques su reemplazo. Tu pool estará más vulnerable hasta que `sdd` también sea reemplazado o retirado si no es parte del pool principal.