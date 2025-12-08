---
title: "De NAS a Homelab: Privacidad, Aprendizaje y Rebelión Tecnológica"
date: 2024-11-24
draft: false
description: "Construye tu propio homelab: privado, seguro y libre."
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
categories:
  - Tecnología
series:
  - Homelab desde cero
libraries:
  - mermaid
image: images/feature2/nas.png
---

## ¿Por qué un homelab y no solo un NAS?

Más que un simple NAS, quiero construir un **homelab**: un espacio para servicios familiares, privados y seguros. Un acto de rebeldía contra la erosión de la privacidad y una forma de recuperar el control sobre nuestros datos. 

### Objetivos:
- **Aprendizaje**: Experimentar con ZFS, virtualización, redes y autohospedaje.
- **Privacidad**: Dejar de depender de servicios cloud que monetizan tus datos.
- **Autonomía**: Tener control total sobre tu infraestructura digital.

---

## Componentes clave

### 🔧 Hardware

| Componente               | Especificación                          | Notas                                                                 |
|--------------------------|----------------------------------------|-----------------------------------------------------------------------|
| **Placa base + CPU**     | Intel N100/N6005 o RK3588 (ARM)        | Equilibrio entre potencia y eficiencia energética.                    |
| **RAM**                  | 32-64 GB DDR4                          | ZFS y máquinas virtuales son ávidas de memoria.                      |
| **Almacenamiento**       | 2x NVMe (256 GB) + 4x HDD (8 TB SATA)  | NVMe para sistema/VM, HDD para almacenamiento masivo con ZFS.         |
| **Fuente de alimentación**| 180W 80+ Platinum                      | Suficiente para picos de arranque (≈77W en operación normal).         |
| **Caja**                 | Jonsbo N1 o alternativa DIY            | Priorizar ventilación y espacio para discos.                          |

#### Opciones de placa base:
1. **RK3588 (ARM)**  
   - [FriendlyElec CM3588](https://www.friendlyelec.com/index.php?route=product/product&product_id=299)  
   ![CM3588](/images/feature1/cm3588.png)  
   *Ventaja: Eficiencia energética. Desventaja: Compatibilidad con software x86.*

2. **Intel N100 (x86)**  
   - [CWWK N100 4xNVMe](https://cwwk.net/products/x86-p5-4-m2-nvme-12th-gen-intel-n100)  
   ![N100](/images/feature2/n100-4nvme.png)  
   - [Topton N100 6xSATA](https://www.toptonpc.com/product/12th-gen-intel-n100-nas-motherboard/)  
   ![N100](/images/feature2/n100-2nvme-6sata.png)  

3. **Intel N6005 (x86)**  
   - [CWWK N6005 6xSATA](https://cwwk.net/products/n5105-n6005nas-demon-board-six-sata3-0-dual-m-2-itx-four-i226-v-nics)  
   ![N6005](/images/feature2/n6005.png)  
   *Aviso: BIOS compleja ([descarga aquí](https://pan.x86pi.cn/BIOS%E6%9B%B4%E6%96%B0/3.NAS%E5%AD%98%E5%82%A8%E7%B1%BB%E4%BA%A7%E5%93%81%E7%B3%BB%E5%88%97BIOS)).*

---

### 📦 Cajas recomendadas
| Modelo               | Enlace                                  | Imagen                          |
|----------------------|----------------------------------------|---------------------------------|
| **Homelabs HL1**     | [Mini-ITX](https://homelabs.club/producto/caja-miniitx-hl1/) | ![HL1](/images/feature2/HL1.png) |
| **Jonsbo N1**        | [6xHDD](https://www.jonsbo.com/en/products/N1.html) | ![Jonsbo](/images/feature2/jobso.png) |
| **DIY (reciclaje)**  | *Ejemplo con caja vieja*               | ![DIY](/images/feature2/walla.png) |

> 💡 *Con creatividad, cualquier caja puede convertirse en un homelab. ¡Incluso con un asa para "escapar" de la nube!*  
> ![Caja con asa](/images/feature2/e71-e73.png)

---

## ⚡ Consumo energético
### Estimación por componente:
| Componente               | Consumo (W) |
|--------------------------|-------------|
| Placa base + CPU         | 15          |
| RAM (2x32GB)             | 10          |
| 2x NVMe                  | 7           |
| 4x HDD SATA              | 40          |
| Red/otros                | 5           |
| **Total (operación)**    | **77 W**    |

**Recomendación**: Fuente de 160-180W para manejar picos (ej. arranque de discos).

---

## 🚀 Más allá del almacenamiento
Este homelab será el corazón de:
- **Servicios familiares**: Nextcloud, Photoprism, Jellyfin.
- **Seguridad**: AdGuard Home, VPN WireGuard.
- **Aprendizaje**: Kubernetes, Proxmox, monitorización con Grafana.

### Filosofía
> *"Gobernanza tecnológica es rebelarse contra el *status quo*: autohospedar es un acto político."*  

---

## Próximos pasos
En futuros posts:
1. Instalación de Proxmox/ZFS.
2. Configuración de redes aisladas.
3. Migración de servicios desde la nube.

¿Te unes a la rebelión? 🚪🔑