---
title: "Wazuh en UGreen"
date: 2026-01-11
draft: false
description: "Guía técnica y las consideraciones críticas para desplegar Wazuh en un UGreen DXP4800 Plus."
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
  - wazuh
  - siem
  - seguridad
  - docker
  - homelab
categories:
  - Seguridad
  - Homelab
  - docker
image: images/feature1/wazuh.png
---

## 🌐 ¿Qué es Wazuh?

**Wazuh** es una plataforma de seguridad de código abierto, gratuita y universalmente reconocida. Funciona como una solución **XDR** (Detección y Respuesta Extendidas) y **SIEM** (Gestión de Eventos e Información de Seguridad) todo en uno.

Wazuh centraliza, analiza y responde a eventos de seguridad de toda tu infraestructura, desde servidores y PCs hasta dispositivos móviles.

### ¿Qué Hace?

*   **Detección de Intrusiones (HIDS):** Monitoriza tus sistemas en busca de actividad sospechosa, cambios no autorizados en ficheros y malware.
*   **Análisis de Logs:** Recopila y analiza logs de cualquier fuente (aplicaciones, sistemas operativos, firewalls) para detectar patrones de ataque.
*   **Monitorización de Integridad de Ficheros (FIM):** Te alerta si ficheros críticos del sistema o de tus aplicaciones son modificados.
*   **Evaluación de Vulnerabilidades:** Detecta software obsoleto o con vulnerabilidades conocidas en tus equipos.
*   **Cumplimiento Normativo:** Ayuda a cumplir con estándares de seguridad como PCI DSS, GDPR o HIPAA mediante reglas predefinidas.
*   **Respuesta Activa:** Puede tomar acciones automáticas, como bloquear una dirección IP en el firewall o detener un proceso malicioso.

## 🛠️ Instalación del servidor Wazuh en Docker

Instalar **Wazuh** en un NAS UGREEN es un salto de nivel importante. No es un contenedor ligero (como Pi-hole o Home Assistant); es un SIEM (Gestión de Eventos e Información de Seguridad) completo que demanda recursos y configuraciones específicas del kernel de Linux.

---

# Despliegue de Wazuh en UGREEN NASync

Guía técnica y las consideraciones críticas para desplegarlo en un **DXP4800 Plus**.

## 1. Consideraciones de Hardware (Crítico)
Wazuh se compone de tres partes: el **Indexer** (basado en OpenSearch), el **Server** y el **Dashboard**.
*   **Memoria RAM:** Wazuh requiere un mínimo de **4GB de RAM libres** solo para el stack. Si tu DXP4800 Plus tiene los 8GB de serie, estarás al límite si tienes más servicios. Se recomienda ampliar a 16GB o 32GB si es posible.
*   **Almacenamiento (IOPS):** Es obligatorio instalarlo en el **Volumen SSD (M.2)**. El Indexer realiza constantes operaciones de lectura/escritura de logs; en discos mecánicos el rendimiento será pésimo y el sistema se volverá inestable.
*   **CPU:** El i5 del DXP4800 Plus es más que suficiente para una pequeña empresa.
  
  Al venir de **Proxmox**, estás acostumbrado al aislamiento rígido (asígnas 4GB a una VM y esos 4GB "desaparecen" del host). En **Docker**, por defecto, los contenedores comparten todo el hardware sin límites, pero para un servicio crítico como **Wazuh**, **debemos limitar los recursos** para evitar que un pico de logs bloquee el NAS entero.

Aquí tienes la configuración óptima para una pequeña empresa (5-25 agentes) en tu **UGREEN DXP4800 Plus**.

### 1. La realidad del hardware: Ampliación de RAM
Antes de pasar al despliegue, una advertencia seria: **Wazuh con 8GB de RAM en el NAS funcionará muy mal**. Entre el sistema operativo UGOS, Docker y el stack de Wazuh, agotarás la memoria, el NAS usará "SWAP" (disco) y el panel irá lentísimo.
*   **Recomendación Pro:** Amplía el NAS a **16GB o 32GB** (usa módulos DDR5 SO-DIMM compatibles). Es una inversión mínima para un SIEM empresarial.

---

### 2. Asignación de Recursos (Docker Compose)

En Docker, los límites se definen en la sección `deploy`. Aquí tienes el reparto de "raciones" recomendado:

| Componente | RAM Mínima | RAM Óptima | CPU (Cores) | Rol |
| :--- | :--- | :--- | :--- | :--- |
| **Wazuh Indexer** | 3 GB | **4 GB** | 2 Cores | El motor de búsqueda (el más pesado). |
| **Wazuh Server** | 1.5 GB | **2 GB** | 1 Core | Análisis de logs y gestión de agentes. |
| **Wazuh Dashboard** | 1 GB | **1.5 GB** | 0.5 Core | Interfaz gráfica. |
| **TOTAL STACK** | **5.5 GB** | **7.5 GB** | **3.5 Cores** | |

---

### 3. Cómo aplicarlo en docker-compose.yml

A diferencia de Proxmox, donde lo haces en la GUI, aquí lo añades al código. Este es un ejemplo de cómo limitar el **Indexer** (el más crítico):

```yaml
services:
  wazuh.indexer:
    image: wazuh/wazuh-indexer:4.7.2
    environment:
      # MUY IMPORTANTE: El "Heap Size" de Java debe ser el 50% de la RAM asignada
      - "OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g" 
    deploy:
      resources:
        limits:
          memory: 4G    # No permite que use más de esto (como en Proxmox)
          cpus: '2.0'   # Máximo 2 núcleos del i5
        reservations:
          memory: 2G    # Garantiza que el NAS le reserve esto al arrancar
    ulimits:            # Requisito específico de OpenSearch
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
```

---

### 4. Gestión del Almacenamiento (Diferencias con Proxmox)

En Proxmox creas un disco virtual (`.qcow2`). En Docker en UGREEN, usas **Bind Mounts** (carpetas directas):

*   **Ubicación:** `/volume1/Docker/wazuh/...`
*   **Asignación de espacio:** Docker no limita el tamaño de la carpeta por defecto. Usará todo el espacio disponible en tu volumen M.2.
*   **Tip de Seguridad (ENS):** Crea una cuota de carpeta en el panel de UGREEN para la carpeta `/Docker` si no quieres que un error de logs llene el SSD por completo.

---

### 5. Comparativa Proxmox vs Docker

| Concepto | Proxmox (VM) | Docker (UGREEN) |
| :--- | :--- | :--- |
| **CPU** | Cores fijos o sockets. | `cpus: '2.0'` (unidades de procesamiento). |
| **RAM** | Estática (la bloquea al iniciar). | Dinámica con `limits` (solo usa lo que necesita). |
| **Disco** | Fichero de disco virtual (.raw/.qcow2). | Carpetas locales del NAS (Filesystem directo). |
| **Kernel** | Cada VM tiene su propio kernel. | Todos comparten el kernel de UGOS (Debian). |


## 2. Preparación del Sistema 
El componente **Wazuh Indexer** (OpenSearch) no arrancará a menos que el sistema operativo UGOS tenga un parámetro del kernel específico configurado.

OpenSearch crea miles pequeños mapas de memoria para gestionar sus índices. Por defecto, Linux (y por tanto el sistema UGOS de UGREEN basado en Debian) viene configurado con un límite de 65,530. OpenSearch necesita al menos 262,144 para arrancar.

### Ajuste de "vm.max_map_count"
  * Debes entrar por **SSH** a tu NAS (actívalo en Panel de Control > Terminal y SNMP) y ejecutar:
```bash
# Para aplicar el cambio inmediatamente
sudo sysctl -w vm.max_map_count=262144

# Para que el cambio persista tras reiniciar el NAS
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```
*Si no haces esto, el contenedor del Indexer entrará en un bucle de reinicio (Bootloop).*
Error en log:
```bash
"max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]"
```
## 3. Método de Instalación: Docker Compose (Portainer)
No intentes instalar Wazuh imagen por imagen desde la interfaz de UGREEN. La complejidad de las redes internas y volúmenes requiere **Docker Compose**.

Para desplegar **Wazuh** de forma profesional en tu UGREEN NASync, vamos a seguir este ejemplo práctico paso a paso. Olvídate de la interfaz "App Center" de UGREEN para esto; usaremos **SSH** para la preparación y **Portainer** para el despliegue.

### Paso 1: Creación de la Estructura de Carpetas (vía SSH o App Archivos)

Necesitamos carpetas persistentes para que, si el contenedor se actualiza, no pierdas los logs ni las alertas.

Entra por SSH y ejecuta:
```bash
# Crear la estructura exacta para el Manager y Dashboard
mkdir -p /volume1/Docker/wazuh/wazuh_indexer_data
mkdir -p /volume1/Docker/wazuh/wazuh_etc
mkdir -p /volume1/Docker/wazuh/wazuh_var
mkdir -p /volume1/Docker/wazuh/certs

# Permisos para el Indexer (Usuario 1000)
sudo chown -R 1000:1000 /volume1/Docker/wazuh/wazuh_indexer_data
```

---

### Paso 2: Generación de Certificados (El paso clave)

Wazuh no arranca sin certificados TLS válidos para cifrar la comunicación entre sus 3 nodos internos. La forma más fácil de hacerlo es usando el generador oficial de Wazuh mediante un contenedor temporal:

1.  **Descarga el configurador:**
    ```bash
    cd /volume1/Docker/wazuh
    curl -sO https://packages.wazuh.com/4.7/config.yml
    ```
2.  **Edita el `config.yml`** (opcional) si quieres cambiar nombres, pero para un "Single Node" el que viene por defecto sirve.
3.  **Ejecuta el generador:**
    ```bash
    docker run --rm -it -v $(pwd)/config.yml:/config.yml -v $(pwd)/certs:/nodedir wazuh/wazuh-certs-generator:4.7.2
    ```
    *Esto llenará tu carpeta `/volume1/Docker/wazuh/certs` con todos los `.pem` necesarios.*

---

### Paso 3: El Stack de Portainer (Docker Compose)

Ahora ve a tu **Portainer > Stacks > Add Stack** y pega este código adaptado para las rutas que creamos antes de tu UGREEN. 

```yaml
version: '3.8'

services:
  wazuh.indexer:
    image: wazuh/wazuh-indexer:4.7.2
    hostname: wazuh.indexer
    restart: always
    environment:
      - OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g # Ajustado para tus 8GB-16GB de RAM
    deploy:
      resources:
        limits:
          memory: 4G
    volumes:
      - /volume1/Docker/wazuh/wazuh_indexer_data:/var/lib/opensearch/data
      - /volume1/Docker/wazuh/certs:/usr/share/opensearch/config/certs:ro
    ports:
      - "9200:9200"

  wazuh.manager:
    image: wazuh/wazuh-manager:4.7.2
    hostname: wazuh.manager
    restart: always
    environment:
      - INDEXER_URL=https://wazuh.indexer:9200
      - INDEXER_USER=admin
      - INDEXER_PASSWORD=TuPasswordSegura123! # CAMBIA ESTO
    deploy:
      resources:
        limits:
          memory: 2G
    volumes:
      - /volume1/Docker/wazuh/wazuh_etc:/var/ossec/etc
      - /volume1/Docker/wazuh/wazuh_var:/var/ossec/var
      - /volume1/Docker/wazuh/certs:/var/ossec/etc/indexer-certs:ro
    ports:
      - "1514:1514/tcp" # Logs de agentes
      - "1514:1514/udp"
      - "1515:1515"     # Registro de agentes
      - "55000:55000"   # API

  wazuh.dashboard:
    image: wazuh/wazuh-dashboard:4.7.2
    hostname: wazuh.dashboard
    restart: always
    environment:
      - INDEXER_URL=https://wazuh.indexer:9200
      - WAZUH_API_URL=https://wazuh.manager:55000
      - INDEXER_USER=admin
      - INDEXER_PASSWORD=TuPasswordSegura123! # DEBE COINCIDIR CON EL INDEXER
    deploy:
      resources:
        limits:
          memory: 1.5G
    volumes:
      - /volume1/Docker/wazuh/certs:/usr/share/wazuh-dashboard/config/certs:ro
    ports:
      - "8443:5601" # Accederás por https://IP-DEL-NAS:8443

networks:
  default:
    name: wazuh_network
    driver: bridge
```

---

### Paso 4: Consideraciones Finales tras el Despliegue

1.  **Primer Inicio:** La primera vez puede tardar hasta 3-5 minutos en levantar todos los servicios. OpenSearch (Indexer) es lento al inicializar los índices en el SSD.
2.  **Acceso:** Abre tu navegador y ve a `https://192.168.1.100:8443` (usa la IP de tu NAS).
    *   **User:** admin
    *   **Pass:** La que pusiste en el YAML (o la por defecto si no la cambiaste en el script de certs).
3.  **Verificación de Agentes:**
    *   Para añadir un PC, ve a "Add Agent" en el Dashboard.
    *   Te dará un comando de PowerShell o Bash. Asegúrate de que la IP que aparece en ese comando sea la IP de tu NAS o el dominio de Tailscale.

### ¿Por qué este método es mejor para el ENS?
Al tener los datos en `/volume1/Docker/wazuh/wazuh_etc`, puedes incluir esa carpeta en tus **copias de seguridad programadas** (Hyper Backup o similar en UGREEN). Si el NAS falla, mueves esas carpetas a otro equipo con Docker, levantas el stack y tienes toda la configuración y evidencias del ENS intactas.

## 4. Puertos Clave a Gestionar
Si vas a monitorizar equipos que están fuera de la red local del NAS (en la misma empresa o sucursales), asegúrate de que estos puertos estén operativos:
*   **1514 (TCP/UDP):** Comunicación de los Agentes (Logs).
*   **1515 (TCP):** Registro de nuevos Agentes.
*   **55000 (TCP):** API de Wazuh.
*   **8443 (o el que asignes):** Para acceder al Dashboard (Panel de control).

## 5. Tips de Mantenimiento en UGREEN

### Tip 1: Logs Rotativos e Índices (Gestión de Almacenamiento)

En una empresa, Wazuh puede recolectar gigabytes de logs diariamente. Si no lo controlas, el SSD M.2 del NAS se llenará y el sistema colapsará. Para el **ENS**, normalmente se exige una retención mínima de logs (ej. 1 año para auditoría), pero no todos tienen que estar "en caliente".

#### Ejemplo Práctico: Política de Retención (Index State Management - ISM)
Wazuh Indexer (OpenSearch) gestiona los datos en "índices". Vamos a crear una política para borrar datos automáticamente tras 90 días:

1.  Entra en el **Wazuh Dashboard** > Menú (hamburguesa) > **Index Management**.
2.  Ve a **Policies** > **Create Policy**.
3.  Usa el editor JSON para crear una política de "Borrado automático":
```json
{
  "policy": {
    "description": "Borrar logs de seguridad tras 90 dias.",
    "default_state": "hot",
    "states": [
      {
        "name": "hot",
        "actions": [],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "90d"
            }
          }
        ]
      },
      {
        "name": "delete",
        "actions": [
          {
            "delete": {}
          }
        ],
        "transitions": []
      }
    ]
  }
}
```
4.  Aplica esta política a los índices `wazuh-alerts-*`. De esta forma, tu NAS siempre tendrá espacio libre.

---

### Tip 2: Alertas por Email y Slack (Notificación de Incidentes)

El **ENS** exige que la organización sea capaz de detectar y **notificar incidentes**. No sirve de nada que el NAS registre un ataque si nadie se entera hasta el lunes.

#### A. Configuración de Email (SMTP)
Debes editar el archivo de configuración del **Wazuh Manager**. En tu UGREEN, lo mapeamos a `/volume1/Docker/wazuh/wazuh_etc/ossec.conf`.

Abre el archivo y busca la sección `<global>`:
```xml
<ossec_config>
  <global>
    <email_notification>yes</email_notification>
    <smtp_server>smtp.gmail.com</smtp_server>
    <email_from>seguridad-nas@tuempresa.com</email_from>
    <email_to>tu-correo@gmail.com</email_to>
    <email_maxperhour>12</email_maxperhour>
    <email_ids_name>Alertas Seguridad UGREEN</email_ids_name>
  </global>

  <alerts>
    <log_alert_level>3</log_alert_level>
    <email_alert_level>10</email_alert_level> <!-- Solo envía email si la alerta es grave (nivel 10+) -->
  </alerts>
</ossec_config>
```
*Nota: Para Gmail/Outlook necesitarás una "Contraseña de Aplicación" si tienes 2FA.*

#### B. Configuración de Slack (Más moderno y efectivo)
Slack es mucho mejor para ver alertas en el móvil en tiempo real.
1.  Crea un **Incoming Webhook** en Slack (o Teams).
2.  Añade este bloque al final de tu `ossec.conf`:
```xml
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/T000000/B000000/XXXXXXXXXXXX</hook_url>
  <level>10</level> <!-- Nivel de alerta mínimo para disparar el mensaje -->
  <alert_format>json</alert_format>
</integration>
```
3.  Reinicia el contenedor de Wazuh Manager desde Portainer para aplicar los cambios.

---

### Tip 3: Para que el NAS te avise si alguien intenta entrar en el propio panel de UGREEN por fuerza bruta

El panel de UGREEN (UGOS) corre sobre el NAS, pero Wazuh corre dentro de un contenedor. Para que Wazuh pueda "leer" los intentos de login fallidos del NAS, necesitamos mapear los logs del sistema operativo host al contenedor del Manager.

### Paso 1: Mapear los Logs del NAS al Contenedor
Debes modificar tu archivo `docker-compose.yml` en Portainer para que el servicio `wazuh.manager` tenga acceso a los logs de autenticación del NAS:

```yaml
services:
  wazuh.manager:
    # ... resto de la configuración ...
    volumes:
      - /var/log/auth.log:/var/log/ugreen_auth.log:ro  # Mapea el log de auth del NAS
      # ... otros volúmenes ...
```
*Nota: En sistemas basados en Debian (como UGOS), `/var/log/auth.log` registra los intentos de SSH y logins del sistema.*

---

### Paso 2: Configurar Wazuh para leer ese archivo
Edita el archivo de configuración de Wazuh (`/volume1/Docker/wazuh/wazuh_etc/ossec.conf`) y añade este bloque para que empiece a monitorizar el nuevo archivo:

```xml
<ossec_config>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/ugreen_auth.log</location>
  </localfile>
</ossec_config>
```

---

### Paso 3: Crear la Regla Personalizada
Wazuh ya tiene reglas para SSH, pero el panel web de UGREEN puede tener un formato de log específico. Vamos a crear una regla que detecte múltiples fallos y dispare una alerta nivel 10 (la que configuramos para Slack/Email).

Edita el archivo de reglas locales: `/volume1/Docker/wazuh/wazuh_etc/rules/local_rules.xml`

```xml
<group name="ugreen,bruteforce,">
  <!-- Regla base: Detectar un fallo de login en el panel UGREEN -->
  <rule id="100001" level="5">
    <decoded_as>syslog</decoded_as>
    <match>web login failed|authentication failure</match>
    <description>Fallo de inicio de sesión en el panel UGREEN.</description>
  </rule>

  <!-- Regla de Fuerza Bruta: 5 fallos en menos de 2 minutos -->
  <rule id="100002" level="10">
    <if_matched_sid>100001</if_matched_sid>
    <same_source_ip />
    <frequency>5</frequency>
    <timeframe>120</timeframe>
    <description>Ataque de fuerza bruta detectado en el panel UGREEN de la empresa.</description>
    <mitigation>active-response</mitigation> <!-- Opcional: para bloquear la IP -->
  </rule>
</group>
```

---

### Paso 4: Ejemplo de Alerta en tu Móvil (Slack/Email)

Una vez reinicies el stack en Portainer, si alguien intenta adivinar la contraseña de tu NAS 5 veces, recibirás una notificación automática:

> **WAZUH Alert**
> **Level:** 10 (Critical)
> **Rule:** 100002 - Ataque de fuerza bruta detectado en el panel UGREEN de la empresa.
> **Location:** (IP del atacante)
> **Description:** 5 login failures from same IP in 120 seconds.
> **Compliance:** ENS op.exp.4 (Detección de intrusión), PCI DSS 10.2.4.

---

**¿Qué hace?**
Cuando Wazuh detecta la fuerza bruta (Regla 100002), envía una orden al NAS UGREEN para que ejecute un script que **bloquee la IP del atacante en el firewall (iptables)** automáticamente durante 24 horas.

1.  **Protección 24/7:** No dependes de que tú veas el email a las 3 de la mañana.
2.  **Evidencia de Cumplimiento:** Para el ENS, no solo estás *monitorizando* (detección), sino que tienes una medida de *protección activa* (reacción ante incidentes).


## 6. Resolución de problemas comunes en UGREEN
*   **Contenedor "Unhealthy":** Generalmente es falta de RAM o no haber aplicado el `vm.max_map_count`.
*   **Permisos de Carpeta:** Si el Indexer falla al escribir, asegúrate de que las carpetas en el NAS tengan permisos totales (`chmod 777` o asignación al usuario Docker) ya que OpenSearch usa un ID de usuario interno (1000).

### Resumen de Mantenimiento Mensual:
*   **Revisar Espacio:** Mira en el panel de UGREEN que el volumen SSD no pase del 80%.
*   **Actualizar Stack:** En Portainer, haz un "Pull and Redeploy" de vez en cuando para tener la última versión de Wazuh (que incluye nuevas reglas de detección de amenazas).
*   **Verificar Agentes:** Entra una vez por semana al Dashboard para ver si algún agente ha dejado de reportar (posible equipo infectado o apagado).
