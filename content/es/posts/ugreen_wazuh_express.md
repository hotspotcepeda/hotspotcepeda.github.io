---
title: "Wazuh en UGreen express"
date: 2026-01-12
draft: false
description: "Desplegar Wazuh en un UGreen DXP4800 Plus git clone."
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

### Wazuh en UGREEN Express 🚀
---

#### Paso 1: Preparación (SSH)
Preparar Kernel y archivos:

```bash
# 1. Ajuste del Kernel (Persistente tras reinicio)
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf

# 2. Directorio de trabajo
cd /volume1/Docker

# 3. Clonar el repositorio oficial (v4.7.2 o la estable que corresponda)
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.2 --depth=1
cd wazuh-docker/single-node
```

#### Paso 2: Generación del Stack
Generador automático de certificados Wazuh.

```bash
# A. Generar certificados TLS (10 segundos)
docker-compose -f generate-certs.yml run --rm generator

# B. Levantar el sistema completo ( 5-10 min )
docker-compose up -d
```
**Acceso:** `https://IP-DEL-NAS:5601`(certificado auto-firmado).

#### Paso 3: Integración con Portainer (Gestión GUI)
1.  **Portainer** > **Stacks**.
2.  Verás el stack `single-node` como **"External"**.
3.  Clic y **Editor**. Portainer te permitirá tomar el control del código. A partir de ahora, gestiona contraseñas o actualizaciones desde aquí, sin tocar más el terminal.

---

#### 💡 Resumen:
```bash
sudo sysctl -w vm.max_map_count=262144
cd /volume1/Docker
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.2 --depth=1
cd wazuh-docker/single-node
docker-compose -f generate-certs.yml run --rm generator
docker-compose up -d
```
**IMPORTANTE:** Las credenciales por defecto están en el archivo `.env`. Por defecto: `admin` / `SecretPassword`.

---
 🚀 De esta manera tienes un **SIEM de grado empresarial** en lo que tarda en hacerse un café.