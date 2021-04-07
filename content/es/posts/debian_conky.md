---
title: "Instalar conky en Debian"
date: 2017-04-15
draft: false
description: "Instalación de conky system monitor en Debian"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- debian
- conky
- linux
categories:
- Linux
series:
- Red comunitaria
image: images/feature1/debian_logo.png
---
{{< notice info >}}
https://github.com/brndnmtthws/conky
{{< /notice >}}
![Configuración de monitor de sistema en escritorio](/images/post/escritorio_conky.png)
```
# apt-get install conky conky-all
# apt-get install curl lm-sensors hddtemp
```
Para autoarranque agregar en programas al inicio:

conky p -15

## start
```
conky -d
```
stop
```
pkill conky
```
restart
```
pkill -HUP conky
```
## Config 

nano /etc/conky/conky.conf
```
background yes
font Zekton:size=8
xftfont Zekton:size=8
use_xft yes
xftalpha 0.4
update_interval 1.0
total_run_times 0
own_window yes
own_window_type override
own_window_transparent yes
own_window_hints undecorated,below,skip_taskbar,skip_pager
double_buffer yes
draw_shades no
draw_outline no
draw_borders no
draw_graph_borders no
minimum_size 220 5
maximum_width 220
default_color ffffff
default_shade_color black
default_outline_color black
alignment top_right
gap_x 10
gap_y 90
no_buffers yes
cpu_avg_samples 2
override_utf8_locale no
uppercase no # set to yes if you want all text to be in uppercase
use_spacer no
TEXT
${font Zekton:style=Bold:pixelsize=42}${alignc}${time %H:%M:%S}${font Zekton:size=8}
lenovo T430s ${hr 1 }
Hostname: $alignr$nodename
IP: $alignr${addr eth0}
Kernel: $alignr$kernel
Uptime: $alignr$uptime
Processes: ${alignr}$processes ($running_processes running)
Load: ${alignr}$loadavg
CPU0 ${alignc} ${freq}MHz ${alignr}(${cpu cpu0}%)
${cpubar cpu0 4}
CPU1 ${alignc} ${freq}MHz ${alignr}(${cpu cpu1}%)
${cpubar 4 cpu1}
CPU2 ${alignc} ${freq}MHz ${alignr}(${cpu cpu2}%)
${cpubar 4 cpu2}
CPU3 ${alignc} ${freq}MHz ${alignr}(${cpu cpu3}%)
${cpubar 4 cpu3}
CPU AVG
${cpugraph}
RAM ${alignr}$mem / $memmax ($memperc%)
${membar 4}
SWAP ${alignr}$swap / $swapmax ($swapperc%)
${swapbar 4}
Maximos de CPU $alignr CPU% MEM%
${top name 1}$alignr${top cpu 1}${top mem 1}
${top name 2}$alignr${top cpu 2}${top mem 2}
${top name 3}$alignr${top cpu 3}${top mem 3}
Maximos de MEM $alignr CPU% MEM%
${top_mem name 1}$alignr${top_mem cpu 1}${top_mem mem 1}
${top_mem name 2}$alignr${top_mem cpu 2}${top_mem mem 2}
${top_mem name 3}$alignr${top_mem cpu 3}${top_mem mem 3}
HDD FREE ${hr 1}${color}
/ : ${alignr}${fs_free /} / ${fs_size /}
${fs_bar 4 /}
eth0 ${hr 1}${color}
Down ${downspeed eth0} k/s ${alignr}Up ${upspeed eth0} k/s
${downspeedgraph eth0 25,107} ${alignr}${upspeedgraph eth0 25,107}
Total ${totaldown eth0} ${alignr}Total ${totalup eth0}
wlan0 ${hr 1}${color}
Intensidad WIFI $alignr ${wireless_link_qual wlan0}%
Down ${downspeed wlan0} k/s ${alignr}Up ${upspeed wlan0} k/s
${downspeedgraph wlan0 25,107} ${alignr}${upspeedgraph wlan0 25,107}
Total ${totaldown wlan0} ${alignr}Total ${totalup wlan0}
BATERIA${hr 1}${color}
${battery_bar BAT0}

${font Arial:style=Bold:size=12}
```