---
title: "Vaultwarden"
date: 2025-06-11
draft: false
description: "Vaultwarden"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
categories:
series:
libraries:
- mermaid
image: images/feature2/Vaultwarden.png
---

# Guía Definitiva: Vaultwarden y Tailscale

Después de un intenso viaje de configuración, aquí se presenta una guía completa sobre cómo montar un entorno `homelab` seguro, privado y accesible desde cualquier lugar. El objetivo final: tener nuestro propio gestor de contraseñas (Vaultwarden) funcionando a la perfección, sin exponer un solo puerto a internet.

Este post resume la arquitectura final, la configuración clave de cada componente y las valiosas lecciones aprendidas durante el proceso.

## 🚀 La Arquitectura Final: El Flujo de la Verdad

El error más común es no entender cómo deben comunicarse los servicios. Después de todo el troubleshooting, la arquitectura correcta es simple y elegante:

**`Tu Dispositivo -> AdGuard (DNS) -> Nginx Proxy Manager (Recepción) -> Vaultwarden (Servicio)`**

En una analogía más clara:

1.  **AdGuard (La Guía Telefónica):** Tu dispositivo pregunta por `subdominio.dominio.com`. AdGuard le da el número de la **Recepción del Edificio**, no el de la oficina interna.
2.  **Nginx Proxy Manager (La Recepción Segura):** Es la única puerta de entrada. Recibe la llamada en un idioma seguro (HTTPS), verifica la identidad (el dominio) y transfiere la llamada a la oficina correcta en un idioma interno (HTTP).
3.  **Vaultwarden (La Oficina Privada):** Hace el trabajo real. No tiene puerta a la calle y solo habla con la recepción.

Con esto claro, ¡vamos a la configuración!

## 🛠️ Componentes del Ecosistema

| Componente            | Rol                                        | Plataforma        | IP Ejemplo       |
| --------------------- | ------------------------------------------ | ----------------- | ---------------- |
| **Proxmox**           | La base de virtualización (Hipervisor)     | Servidor Físico   | N/A              |
| **Cloudflare**        | Gestión de dominio y API para certificados | Servicio Externo  | N/A              |
| **AdGuard Home**      | Servidor DNS local y bloqueo de anuncios   | Contenedor LXC    | `192.168.100.110` |
| **Nginx Proxy Mgr**   | Reverse Proxy y gestión de SSL             | Contenedor LXC    | `192.168.100.183` |
| **Vaultwarden**       | Servidor de contraseñas (Bitwarden)        | Contenedor LXC    | `192.168.100.182` |
| **Tailscale**         | VPN segura para acceso remoto (Overlay Net) | Todos los equipos | IP de Tailscale  |

## ⚙️ Guía de Configuración Clave

Aquí no se detallará la instalación básica de cada LXC (que se ha hecho con los scripts de https://tteck.github.io/) sino los puntos de configuración críticos que hicieron que todo funcionara.

### 1. Cloudflare: El Dominio

*   **Rol:** Únicamente gestionar el nombre del dominio y subdominios. No usaremos su proxy.
*   **Configuración Crítica:** Crea los registros DNS de tipo `A` para tus servicios (subdominios, ej. `vault`, `adguard`, `npm`) apuntando a sus IPs locales. Asegúrate de que el **Estado del Proxy** esté en **"Solo DNS"** (nube gris). Esto es fundamental.

### 2. AdGuard Home: La Guía Telefónica

*   **Rol:** Resolver nuestros dominios locales a las IPs correctas.
*   **Configuración Crítica:** `Filters` -> `DNS rewrites`.
    *   Crea una reescritura para tu servicio.
    *   **Dominio:** `vault.dominio.com`
    *   **IP de Respuesta:** **¡DEBE APUNTAR AL REVERSE PROXY (NPM)!**.
        ```
        192.168.100.183
        ```

### 3. Vaultwarden: La Oficina Privada

*   **Rol:** Almacenar de forma segura nuestras contraseñas.
*   **Configuración Crítica:** El fichero `/opt/vaultwarden/.env`. Debe contener estas cuatro variables para funcionar detrás de un reverse proxy:

    ```ini
    # Fichero: /opt/vaultwarden/.env

    # 1. Tu dominio completo con HTTPS. Obligatorio para los WebSockets.
    DOMAIN=https://subdominio.dominio.com

    # 2. Habilita el soporte de WebSockets para que la web no se quede en "LOADING".
    WEBSOCKET_ENABLED=true

    # 3. El token para acceder a la página de administración (/admin).
    ADMIN_TOKEN=TU_TOKEN_SEGURO_AQUÍ

    # 4. ¡CRUCIAL! Para que Vaultwarden acepte conexiones de otros contenedores (NPM).
    ROCKET_ADDRESS=0.0.0.0
    ```
    Después de editar, reinicia el servicio: `systemctl restart vaultwarden`.

### 4. Nginx Proxy Manager: La Recepción Segura

*   **Rol:** Recibir el tráfico HTTPS, desencriptarlo y pasarlo a Vaultwarden.
*   **Configuración Crítica:** Crear un `Proxy Host`.
    *   **Pestaña Details:**
        *   Domain Names: `vault.dominio.com`
        *   Scheme: `http`
        *   Forward Hostname / IP: `192.168.100.182` (La IP de Vaultwarden)
        *   Forward Port: `8000`
    *   **Pestaña SSL:**
        *   Selecciona tu certificado SSL (puedes usar el desafío DNS de Cloudflare para obtenerlo).
        *   Activa `Force SSL` y `HTTP/2 Support`.
    *   **Pestaña Advanced (Opcional):** El toggle de "Websocket Support" a veces no es suficiente. Usar esta configuración personalizada es más robusto.

        ```nginx
        location / {
            proxy_pass http://192.168.100.182:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        location /notifications/hub {
            proxy_pass http://192.168.100.182:8000;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
        ```

### 5. Tailscale: La Cúpula de Acceso Remoto

*   **Rol:** Crear una red privada virtual (VPN) segura para acceder a nuestro homelab desde cualquier lugar del mundo, sin abrir puertos y funcionando perfectamente detrás de CG-NAT.
*   **Configuración:**
    1.  Crea una cuenta en Tailscale.
    2.  Instala el cliente de Tailscale en tus dispositivos (PC, móvil) y en los servidores a los que quieras acceder (¡incluso puedes instalarlo en el contenedor de NPM o en una VM separada!).
    3.  Activa **Magic DNS** en la consola de administración de Tailscale si quieres acceder a tus servicios por su nombre de host simple.

Con Tailscale activo, puedes acceder a `https://subdominio.dominio.com` desde tu móvil en una red 4G como si estuvieras en casa, de forma totalmente segura.

## 💡 Lecciones Aprendidas en el Camino (Troubleshooting)

Este setup es robusto, pero el camino para llegar a él estuvo lleno de pistas falsas. Estos fueron los problemas más destacados:

1.  **Firewall de OPNsense/pfSense:** La causa inicial fue un **proxy transparente** que interceptaba el tráfico HTTPS, causando `ERR_CONNECTION_REFUSED`. Atención a los firewall y reglas residuales.
2.  **DNS Rebinding Protection:** Una función de seguridad en routers avanzados que evita que dominios públicos resuelvan a IPs privadas. Se soluciona añadiendo tu dominio a una lista de excepciones.
3.  **DNS del Navegador vs. Sistema:** Los navegadores modernos con **"DNS Seguro" (DoH)** pueden ignorar tu DNS local (AdGuard). Desactivarlo o usar el fichero `hosts` de Windows fue clave para diagnosticar.
4.  **Configuración de Vaultwarden:** La variable `ROCKET_ADDRESS=0.0.0.0` es imprescindible para que el servicio acepte conexiones externas a su contenedor. Si se queda en `127.0.0.1`, nadie excepto él mismo puede hablarle.
5.  **El Destino del DNS Rewrite:** El error más simple. La reescritura en AdGuard **siempre** debe apuntar al Reverse Proxy (NPM), no al servicio final (Vaultwarden).
6.  **Inprescindible contratar un dominio** En verdad es justo y necesario **Cloudflare** es una opción económica y solida.

Espero que este post sirva de ayuda como una guía clara y una fuente de motivación.
Montar un homelab privado y seguro es un viaje de aprendizaje increíblemente satisfactorio.

---