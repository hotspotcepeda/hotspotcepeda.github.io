---
title: "Wazuh: Tu Centro de Seguridad Open Source (Guía Completa)"
date: 2024-12-06
draft: false
description: "Aprende qué es Wazuh, cómo funciona y cómo instalar el servidor en Proxmox LXC y los agentes en Windows y Android para una seguridad total."
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
  - wazuh
  - siem
  - xdr
  - seguridad
  - proxmox
  - homelab
categories:
  - Seguridad
  - Homelab
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

## 🛠️ Instalación del Servidor Wazuh en Proxmox LXC

Vamos a instalar la pila completa de Wazuh (Indexer, Server y Dashboard) en un único contenedor LXC, una opción eficiente y potente para un homelab.

### 1. Preparar el Contenedor LXC

1.  **Crea un Contenedor LXC en Proxmox:**
    *   **Plantilla:** Ubuntu 22.04 (Jammy) es una opción excelente y recomendada.
    *   **Recursos:** Wazuh consume recursos, especialmente el Indexer. Asigna como mínimo:
        *   **CPU:** 4 Cores
        *   **RAM:** 4 GB (8 GB es ideal)
        *   **Disco:** 30-50 GB (dependiendo de cuántos logs vayas a almacenar)
    *   **Networking:** Asigna una IP estática para facilitar la gestión.

2.  **Accede al Contenedor y Actualiza:**
    ```bash
    # Entra a la consola del CT
    pct enter ID_DEL_CT

    # Actualiza el sistema
    apt update && apt upgrade -y
    ```

### 2. Instalación de Wazuh (Método Asistido)

Wazuh proporciona un script que simplifica enormemente la instalación "todo en uno".

1.  **Descarga el Asistente de Instalación:**
    ```bash
    curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
    ```

2.  **Ejecuta el Asistente:**
    ```bash
    bash wazuh-install.sh -a
    ```
    El flag `-a` significa "all-in-one" (todo en uno). El script se encargará de instalar Wazuh Indexer, Wazuh Server y Wazuh Dashboard, y de configurarlos para que se comuniquen entre sí.

3.  **¡Ten Paciencia!** Este proceso puede tardar entre 15 y 30 minutos. Al final, el script te mostrará un resumen con las credenciales de administrador (usuario `admin`). **¡Guarda esta contraseña en un lugar seguro!**

4.  **Verifica los Servicios:**
    ```bash
    systemctl status wazuh-indexer wazuh-manager wazuh-dashboard
    ```
    Los tres servicios deberían estar `active (running)`.

¡Y listo! Ya puedes acceder a tu interfaz de Wazuh en `https://IP_DEL_LXC`.

## 💻 Instalación del Agente en Windows

Ahora vamos a conectar un PC con Windows a nuestro servidor Wazuh.

1.  **Descarga el Agente:** Ve a la página oficial de descargas de Wazuh y descarga el agente para Windows.

2.  **Instalación Gráfica:**
    *   Ejecuta el instalador.
    *   Durante la instalación, te pedirá la **dirección del servidor Wazuh**. Introduce la IP de tu contenedor LXC (ej. `192.168.100.101`).
    *   Deja el resto de opciones por defecto y finaliza la instalación.

3.  **Configuración y Arranque:**
    *   El agente se instalará como un servicio de Windows. Para gestionarlo, abre PowerShell como **Administrador**.
    *   Inicia el servicio:
        ```powershell
        net start WazuhSvc
        ```

4.  **Verificación:**
    *   En tu **Dashboard de Wazuh**, después de unos minutos, ve a la sección `Agents`. Verás tu nuevo agente de Windows conectado y reportando datos.

## 📱 Instalación del Agente en Android

Sí, también puedes monitorizar tu dispositivo Android para detectar aplicaciones maliciosas y configuraciones inseguras.

1.  **Descarga la APK:**
    *   Desde tu teléfono Android, ve a la [página de GitHub de Wazuh para Android](https://github.com/wazuh/wazuh-android-agent/releases).
    *   Descarga el fichero `.apk` de la última versión. Es posible que tengas que permitir la instalación desde fuentes desconocidas.

2.  **Configuración de la App:**
    *   Abre la aplicación de Wazuh.
    *   Te pedirá la **IP del servidor** y el **puerto**.
        *   **IP del Servidor:** La IP de tu servidor Wazuh (ej. `192.168.88.101`).
        *   **Puerto:** `1514` (es el puerto por defecto para la comunicación de los agentes).
    *   **Autenticación:** Puedes dejar la contraseña vacía para que el servidor registre el agente automáticamente (si la configuración por defecto lo permite).
    *   Guarda la configuración y pulsa **"Start"**.

3.  **Permisos:** La aplicación te pedirá varios permisos para poder monitorizar el estado del teléfono. Concédeselos para que pueda funcionar correctamente.

4.  **Verificación:**
    *   Al igual que con el agente de Windows, tu dispositivo Android aparecerá en la lista de agentes del Dashboard de Wazuh pasados unos minutos.

Con estos pasos, has creado un centro de mando de seguridad centralizado y potente. Ahora puedes empezar a explorar las reglas, crear alertas personalizadas y tener una visibilidad completa de lo que ocurre en tu red.  🛡️


### Instalación del Agente en Debian

1.  **Pre-instalación de Dependencias:**  `lsb-release` primero. Esto evita cualquier error con `dpkg`.
2.  **Instalación del Agente:**  `dpkg` con las variables de entorno, incluyendo un `WAZUH_AGENT_NAME` único y descriptivo (`NOMBRE-DESCRIPTIVO`).
3.  **Activación y Arranque del Servicio:**
    *   `sudo systemctl enable wazuh-agent` para asegurarte de que sobreviva a los reinicios.
    *   `sudo systemctl start wazuh-agent` para iniciarlo en el momento.
4.  **Estado Final:** El último `systemctl status wazuh-agent` muestra claramente `Active: active (running)` y todos los sub-procesos de Wazuh iniciados correctamente.

---

### Comandos Aislados Debian/Ubuntu.

**Paso 1: Actualizar la lista de paquetes**
```bash
sudo apt-get update
```

**Paso 2: Instalar la dependencia `lsb-release` (si no está presente)**
```bash
sudo apt-get install -y lsb-release
```

**Paso 3: Descargar e instalar el agente Wazuh**
*(Recuerda cambiar `WAZUH_AGENT_NAME` por el nombre de la nueva máquina)*
```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.12.0-1_amd64.deb
```
```bash
sudo WAZUH_MANAGER='192.168.88.188' WAZUH_AGENT_GROUP='center' WAZUH_AGENT_NAME='nombre-del-nuevo-agente' dpkg -i ./wazuh-agent_4.12.0-1_amd64.deb
```

**Paso 4: Habilitar el servicio para que inicie automáticamente**
```bash
sudo systemctl enable wazuh-agent
```

**Paso 5: Iniciar el servicio del agente**
```bash
sudo systemctl start wazuh-agent
```

**Paso 6: Verificar el estado del servicio**
```bash
systemctl status wazuh-agent
```