---
title: "Banner ssh"
date: 2016-03-03
draft: false
description: "Banner ssh linux"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- debian
- ssh
categories:
- red
series:
- Red comunitaria
image: images/feature1/debian_logo.png
---
{{< notice info >}}
Este banner bash requiere de:
```
# apt-get install uprecords hddtemp lm-sensors
```
{{< /notice >}}

```console
#!/bin/bash
$>> _
txtblk='\e[0;30m' # Black - Regular
txtred='\e[0;31m' # Red
txtgrn='\e[0;32m' # Green
txtylw='\e[0;33m' # Yellow
txtblu='\e[0;34m' # Blue
txtpur='\e[0;35m' # Purple
txtcyn='\e[0;36m' # Cyan
txtwht='\e[0;37m' # White
bldblk='\e[1;30m' # Black - Bold
bldred='\e[1;31m' # Red
bldgrn='\e[1;32m' # Green
bldylw='\e[1;33m' # Yellow
bldblu='\e[1;34m' # Blue
bldpur='\e[1;35m' # Purple
bldcyn='\e[1;36m' # Cyan
bldwht='\e[1;37m' # White
unkblk='\e[4;30m' # Black - Underline
undred='\e[4;31m' # Red
undgrn='\e[4;32m' # Green
undylw='\e[4;33m' # Yellow
undblu='\e[4;34m' # Blue
undpur='\e[4;35m' # Purple
undcyn='\e[4;36m' # Cyan
undwht='\e[4;37m' # White
bakblk='\e[40m'   # Black - Background
bakred='\e[41m'   # Red
bakgrn='\e[42m'   # Green
bakylw='\e[43m'   # Yellow
bakblu='\e[44m'   # Blue
bakpur='\e[45m'   # Purple
bakcyn='\e[46m'   # Cyan
bakwht='\e[47m'   # White
txtrst='\e[0m'    # Text Reset
clear
echo ""
echo -e "\033[44m                           Welcome server Ayto Cepeda guifi.net!                          "
echo -en "\033[1;32m
\033[0;36m+++++++++++++++++: \033[0;37m${undylw}System Data\033[0;36m :++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+  \033[0;37mHostname \033[0;36m =  \033[1;32m`hostname`
\033[0;36m+   \033[0;37mAddress \033[0;36m =  \033[1;32mCepeda La Mora, Avila
\033[0;36m+    \033[0;37mKernel \033[0;36m =  \033[1;32m`uname -r`
\033[0;36m+    \033[0;37mUptime \033[0;36m =\033[1;32m`uptime | awk '{ $1=$2=$(NF-6)=$(NF-5)=$(NF-4)=$(NF-3)=$(NF-2)=$(NF-1)=$NF=""; print}'`
\033[0;36m+    \033[0;37mMemory \033[0;36m =  \033[1;32m`free -m | awk 'NR==2 {print $2}'` MB
\033[0;36m+\033[0;37mFree Memory \033[0;36m=  \033[1;32m`free -m | awk 'NR==3 {print $4}'` MB
\033[0;36m++++++++++++++++++: \033[0;37m${undylw}User Data\033[0;36m :+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+  \033[0;37mUsername \033[0;36m =  \033[1;32m`whoami`
\033[0;36m+++++++++++: \033[0;31m${undred}Maintenance Information\033[0;36m :++++++++++++++++++++++++++++++++++++++++++++++++++++
"



echo -e "\033[0;37msessions \033[0;36m \033[1;32m`w`"
echo -e "\033[0;37mreboots \033[0;36m  \033[1;32m`uprecords`"
echo -e "\033[0;37mhddtemp \033[0;36m  \033[1;32m`hddtemp /dev/sda`"
echo -e "\033[0;37msensors \033[0;36m  \033[1;32m`sensors`"
echo -e "\033[0;37mcat /var/log/auth* | grep Failed #ver los fail SSH"
echo -e "\033[0;36m++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\033[0m"

echo -e "\033[34mEn la calle tenemos:"
METRIC=1  # 0 for F, 1 for C
# Fill in form to find your weather code here: 
# http://netweather.accuweather.com/signup-page2.asp
# If code has a space remove it or replace it with %20 or a dash; -
LOCCOD="EUR|ES|GM002|CEPEDA-LA-MORA"  #Example: NAM|MX|MX009|MEXICO-CITY

if [ -z $1 ] && [ -x $LOCCOD ] ; then
        echo
        echo "USAGE: $0 [locationcode]"
        echo
        exit 0;
elif [ ! -z $1 ] ; then
        LOCCOD=$1
fi

curl -s http://rss.accuweather.com/rss/liveweather_rss.asp\?metric\=${METRIC}\&locCode\=$LOCCOD \
| sed -n '/Currently:/ s/.*: \(.*\): \([0-9]*\)\([CF]\).*/\2°\3, \1/p'

echo -e "\e[0m"
```
Queda cosa así:
![Banner ssh debian](/gallery/red/mega_banner.png)