---
title: "DDNS Change IP MikroTik script"
date: 2016-11-13
draft: false
description: "Script para renovar IP DDNS MikroTik"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- mikrotik
- debian
- ddsns
categories:
- red
series:
- Red comunitaria
image: images/feature1/mikrotik.png
---
## Script
```
/system/script new and paste:
```
```
# Written by Sam Norris, ChangeIP.com
# Copyright ChangeIP.com 2009
# For support send mail to Support@ChangeIP.com
#
# 2009-06-22 RouterOS 3.25 Tested
# 2009-10-05 RouterOS 4.01rc1 Tested
#
# OVERVIEW:         %
#  This script will update a ChangeIP.com dynamic dns hostname
#  with an ip address located directly on an interface.
#                   %
# NOTES:            %
#  IF THIS SCRIPT DOES NOT PRODUCE ANY OUTPUT PLEASE COPY AND PASTE IT
#  AGAIN.  THERE PROBABLY IS A LINE BREAK IN THE WRONG PLACE! Once you
#  have created this script and tested that it works by running it
#  manually you can schedule it to run every few minutes.
#                   %
# CONFIGURATION FIELD DEFINITIONS:
#  ddnsuser:  Enter your ChangeIP.com user id.
#  ddnspass:  Enter your ChangeIP.com password.
#  ddnshost:  Enter the hostname (www.example.com) to update.
#  ddnsinterface:  Enter an interface name to watch - case sensative.
#                   %
#                   %
#                   %
#                   %
#               %   %   %
#                %  %  %
#                 % % %
#                   %
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# EDIT YOUR DETAILS / CONFIGURATION HERE
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
:global ddnsuser "user"
:global ddnspass "pass"
:global ddnshost "ddnsname.net"

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# END OF USER DEFINED CONFIGURATION
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:global ddnssystem ("mt-" . [/system package get [/system package find name=system] version] )
/tool fetch url="http://myip.dnsomatic.com/" mode=http dst-path=mypublicip.txt
:global ddnsip [file get mypublicip.txt contents ]
:global ddnslastip



:if ([ :typeof $ddnslastip ] = "nothing" ) do={ :global ddnslastip 0.0.0.0/0 }

:if ([ :typeof $ddnsip ] = "nothing" ) do={
:log info ("DDNS: No ip address present on contents , please check.")

} else={

  :if ($ddnsip != $ddnslastip) do={

    :log info "DDNS: Sending UPDATE!"
    :log info [ :put [/tool dns-update name=$ddnshost address= $ddnsip key-name=$ddnsuser key=$ddnspass ] ]
    :log info $ddnsip
    :global ddnslastip $ddnsip

  } else={ 

    :log info "DDNS: No changes necessary."

  }

}
```
## Schedule
```
/system/schedule Interval: 00:10:00
```