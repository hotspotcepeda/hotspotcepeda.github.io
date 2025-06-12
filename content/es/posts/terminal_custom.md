---
title: "Terminal custom"
date: 2024-11-28
draft: false
description: "Neofetch y Oh My Zsh"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags: 
- consola

image: images/feature1/debian_logo.png
---
# *Neofetch*
## **Instalación**
Neofetch es una herramienta ligera y obsoleta que muestra información del sistema junto a un logotipo ASCII.

Actualiza los paquetes del sistema con:
> *sudo apt update && sudo apt upgrade -y*

Instala Neofetch desde los repositorios oficiales:
> *sudo apt install -y neofetch*

Para ejecutarlo, simplemente escribe:
> *neofetch*

\---
## **Configuración**
Para personalizar Neofetch, edita su archivo de configuración ubicado en:
>cp ${HOME}/.config/neofetch/config.conf_ORI ${HOME}/.config/neofetch/config.conf *

>*~/.config/neofetch/config.conf
o
${HOME}/.config/neofetch/config.conf*

Si deseas que Neofetch se ejecute automáticamente al abrir una nueva terminal, añade el comando neofetch al final del archivo de configuración de tu shell:
> *neofetch*

En bash:
> *nano ~/.bashrc*

En zsh:
> *nano ~/.zshrc*

Guarda los cambios y recarga la configuración con:
> *source ~/.bashrc*

o
> *source ~/.zshrc*

Si necesitas restaurar la configuración original, elimina el archivo de configuración y ejecuta Neofetch nuevamente:
> *rm ~/.config/neofetch/config.conf*
>
> *neofetch*

# *ZSH*
## **Instalación**
Oh My Zsh es una herramienta de gestión de configuraciones para el shell Zsh, diseñada para facilitar y mejorar el uso del terminal. Con el soporte de una amplia variedad de plugins, temas y configuraciones, Oh My Zsh permite personalizar tu experiencia en el terminal de manera sencilla y eficiente.
> *sudo apt update && sudo apt upgrade -y*
> *sudo apt install zsh*

Configurar Zsh como shell predeterminado:
> *chsh -s $(which zsh)*
## **Instalación de Oh My Zsh:**

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
O wget
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"*
```
Configuración de Plugins de Autocompletar

Activación del Plugin de Autocompletar:

Oh My Zsh viene con un plugin de autocompletado que se puede activar editando el archivo de configuración .zshrc:
> *nano ~/.zshr*

Dentro del archivo, encuentra la línea plugins=(git) y añade zsh-autosuggestions y zsh-syntax-highlighting para habilitar el autocompletado y resaltado de sintaxis:

>*plugins=(git zsh-autosuggestions zsh-syntax-highlighting)*

Instalación del Plugin zsh-autosuggestions: Clona el repositorio del plugin en el directorio de plugins de Oh My Zsh:

>*git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions*

Instalación del Plugin zsh-syntax-highlighting: Clona el repositorio del plugin en el directorio de plugins de Oh My Zsh:

>*git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting*

Aplicar los Cambios: Después de añadir los plugins y clonar los repositorios, aplica los cambios reiniciando la terminal o ejecutando:

>*source ~/.zshrc*

 😊
