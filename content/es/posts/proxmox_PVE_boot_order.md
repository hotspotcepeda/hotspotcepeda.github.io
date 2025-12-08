---
title: "Orquestación de Arranque PVE"
date: 2025-12-08
draft: false
description: "Orquestación del Orden de Arranque en Proxmox VE"
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
***

# Optimización y Orquestación del Orden de Arranque en Proxmox VE

**Fecha:** 08 de Diciembre de 2025
**Categoría:** SysAdmin / Proxmox / HomeLab
**Estado:** Implementado

## 1. El Problema: La "Tormenta de Arranque"
En un entorno de virtualización donde conviven servicios críticos de infraestructura (Router, DNS, VPN) con aplicaciones de usuario (Domótica, Nube), el orden de encendido es vital.

Tras un corte de energía o un mantenimiento, si todos los contenedores (LXC) y máquinas virtuales (VM) intentan arrancar simultáneamente, ocurren dos problemas:
1.  **Saturación de I/O y CPU:** El host se ralentiza intentando levantar todo a la vez.
2.  **Fallos de Dependencia:** Home Assistant falla porque no hay red; Nextcloud falla porque no resuelve el DNS; la VPN falla porque el Router aún no ha negociado la IP pública.

## 2. Diagnóstico: Visualización del Estado Actual
Para auditar la configuración actual sin ir una por una en la interfaz web, utilicé el siguiente *one-liner* en la shell del nodo. Esto lista todos los servicios, si tienen activado el arranque automático (`onboot`) y su configuración de orden (`startup`).

**Script de Auditoría:**
```bash
# Listar LXCs
pct list | awk '{print $1, $4}' | while read id name; do 
  [ "$id" != "VMID" ] && echo "LXC $id ($name): $(pct config $id | grep -E 'onboot|startup')"
done

# Listar VMs
qm list | awk '{print $1, $2}' | while read id name; do 
  [ "$id" != "VMID" ] && echo "VM  $id ($name): $(qm config $id | grep -E 'onboot|startup')"
done
```

## 3. La Estrategia: Anillos de Prioridad
Se ha diseñado una estrategia de arranque escalonado en 5 niveles (de 0 a 5), asegurando que la capa inferior esté estable antes de iniciar la siguiente.

| Prioridad | Nivel | Servicios | Retardo (Delay) | Justificación |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **Core Network** | OPNsense (Router) | 60 seg | Necesita tiempo para negociar WAN/DHCP. |
| **2** | **Infraestructura** | Tailscale (VPN), AdGuard (DNS) | 5 seg | Resolución de nombres y acceso remoto. |
| **3** | **Acceso** | Nginx Proxy Manager | 5 seg | Enrutamiento inverso de dominios. |
| **4** | **Aplicaciones** | Home Assistant, Nextcloud, Vaultwarden | 10 seg | Apps de usuario final. Requieren DB y Red estables. |
| **5** | **Monitorización** | Wazuh, Uptime Kuma | 5 seg | Vigilancia. Arrancan al final para evitar falsos positivos. |

## 4. Implementación
Se aplicaron las configuraciones mediante CLI (`qm set` para VMs y `pct set` para LXC) para agilizar el proceso y evitar errores manuales.

### Nivel 1: El Router (OPNsense)
El router es el Gateway de la red. Debe ser el primero absoluto. Le damos 60 segundos de margen para estabilizar la conexión a internet.
```bash
qm set 1110 -onboot 1 -startup order=1,up=60
```

### Nivel 2: Servicios de Red (DNS y VPN)
Una vez el router está vivo, levantamos el DNS y la VPN.
```bash
pct set 100 -onboot 1 -startup order=2,up=5  # Tailscale
pct set 110 -onboot 1 -startup order=2,up=5  # AdGuard
```

### Nivel 3: Proxy Inverso
```bash
pct set 8300 -onboot 1 -startup order=3,up=5 # NPM
```

### Nivel 4: Aplicaciones Críticas
Aquí residen los servicios pesados. Home Assistant tiene un retardo extra (`up=10`) para asegurar que su base de datos interna cargue bien.
```bash
qm set 188 -onboot 1 -startup order=4,up=10  # Home Assistant OS
pct set 8200 -onboot 1 -startup order=4,up=5 # Vaultwarden
qm set 101 -onboot 1 -startup order=4,up=5   # Nextcloud
```

### Nivel 5: Monitorización
```bash
pct set 8800 -onboot 1 -startup order=5,up=5 # Wazuh
pct set 8900 -onboot 1 -startup order=5      # Uptime Kuma
```

## 5. Resultado Final y Validación
Tras aplicar los cambios, el sistema presenta la siguiente configuración de arranque, garantizando una recuperación de desastres limpia y autónoma.

```text
LXC 100 (Tailscale): startup: order=2,up=5
LXC 110 (AdGuard):   startup: order=2,up=5
LXC 8300 (NPM):      startup: order=3,up=5
VM  1110 (OPNsense): startup: order=1,up=60
VM  188 (HAOS):      startup: order=4,up=10
VM  101 (Nextcloud): startup: order=4,up=5
LXC 8200 (Vault):    startup: order=4,up=5
LXC 8800 (Wazuh):    startup: order=5,up=5
LXC 8900 (Kuma):     startup: order=5
```

### Cronología de arranque simulada (T=0 Recuperación de energía)
*   **T+00:00:** Arranca Host Proxmox.
*   **T+00:01:** Inicia **OPNsense**. (Espera 60s).
*   **T+01:01:** Inician **Tailscale** y **AdGuard**. (Red operativa y accesible remotamente).
*   **T+01:06:** Inicia **Nginx Proxy Manager**. (Dominios operativos).
*   **T+01:11:** Inician **Home Assistant** y resto de Apps. (Base de datos y red estables).
*   **T+01:21:** Inicia **Monitorización**.

Con esta configuración, el servidor es capaz de recuperarse de un apagado total sin intervención humana y sin errores de servicios caídos por falta de dependencias.
´´´
