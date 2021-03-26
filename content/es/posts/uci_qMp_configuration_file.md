---
title: "UCI qMp archivo de configuración"
date: 2015-10-26
draft: false
description: "UCI qMp archivo de configuración"
hideToc: false
enableToc: true
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags:
- OpenWRT
- qMp
categories:
- red
series:
- Red comunitaria
image: images/feature1/qmp_logo.png
---
{{< notice info >}}
**¿Que qué es qMp?** Visite http://qmp.cat/Inicio para conocerlo. 
Esta es la página de desarrollo proyecto https://dev.qmp.cat/projects/qmp  **🙌**
{{< /notice >}}
#### Este es el archivo de configuración qMp
``` bash
/etc/config/qmp
	Node configuration:
		config 'qmp ' 'node '
			option 'community_node_id '	'901 '  # An number identifying the node. In this case node name will be: "qmp901". If not specified it will be auto-configured based on the last byte of the MAC address.
			option 'key '			'/tmp/ qmp_key ' # Path to file containing the QMP key. It is a hash generated randomly on each boot used by some QMP applications for security purposes.
	Interfaces
		config 'qmp ' 'interfaces '
			option 'lan_devices '	'eth0 wlan1 '  # Interfaces used for nal client connection, a bridge between them will be created and the DHCP server will be enabled.
			option 'wan_device '	'eth1 '  #  Interface connection with another network. A DHCP client will be enabled in this interface to automatically get the network conguration.
			option 'mesh_devices '	'wlan0 wlan2 '  # Interfaces used for meshing. Several VLAN will be created and routing protocols will be enabled.
	Networks
		config 'qmp ' 'networks '
			option 'dns '			'141.1.1.1 '  # Nameservers used for the device and final clients
			option 'lan_address '		'172.30.22.1 ' # lan address: IPv4 address used for final client bridge interface (see 7.3.2)
			option 'lan_netmask '		'255.255.255.0 ' #  lan netmask: IPv4 netmask used for final client bridge interface
			option 'niit_prefix96 '		'0:0:0:0:0: ffff ' # niit prefix96: Niit is used for 4in6 tunneling, see 7.2.4
			option 'mesh_protocol_vids '	'olsr6 :1 bmx6 :2' # VLAN tag suffix added for routing protocol interfaces (mesh vid - offset + mesh protocol vids)
			option 'mesh_vid_offset '	'10' # VLAN tag offset used for routing protocol interfaces (e. eth0.10)
			option 'olsr6_mesh_prefix48 '	'fd01 :0:0 ' # First 48 bits used for IPv6 calculation, see IP structure chapter
			option 'bmx6_mesh_prefix48 '	'fd02 :0:0 '
			option 'babel_mesh_prefix48 '	'fd03 :0:0 '

		      option 'netserver ' '1'
	Wireless
	General section
		config 'qmp ' 'wireless '
			option 'driver '	'mac80211 ' # Which driver to use: mac80211 or madwi
			option 'country '	'ES ' # country: Country code to use for WiFi devices
			option 'bssid '		'02: CA:FF:EE:BA:BE ' # bssid: BSSID to use for Ad-Hoc devices
	Specific device section looks like that:
		config 'wireless '
			option 'mode '		'adhoc ' # WiFi supported modes are "adhoc", "ap" or "none"
			option 'name '		'qMp ' # WiFi SSID to use in this device
			option 'txpower '	'20' # txpower: Transmit power for this device
			option 'channel '	'48+ ' # channel: Channel for this device. If +/- added, 802.11n in mode HT40 will be used
			option 'mac '		'00: AA:BB:CC:DD:EE ' # mac: MAC identify the device
			option 'device '	'wlan0 ' # device: Interface name used for this device
	Tunnels
		config 'qmp ' 'tunnels '
			option 'search_ipv4_tunnel '	'0.0.0.0/0 '
			option 'offer_ipv6_tunnel '	'::/0 '
	
Autoconfiguration system:
	/qmp_autoconf		 # se ejecuta solo en el primer arranque, si ya existe no se ejecuta 
	/etc/init.d/qmp_autoconf # a init script 
				 # pasos del autoconf 	1. Check if control file exist
							2. Configure wireless devices
							3. Configure networking
							4. Restart affected services
							5. Create control file 

Wireless config
	/etc/qmp/qmp_wireless.sh # dos funciones
	1 qmp_configure_wifi_initial # 
		config 'wireless '
			option 'mode ' 'adhoc '
			option 'name ' 'qMp -MESH '
		config 'wireless '
			option 'mode ' 'ap '
			option 'name ' 'qMp -AP '
	2 qmp_configure_wifi # in /etc/qmp/templates 
					wireless .# QMP_DEVICE =wifi - device
					wireless .# QMP_DEVICE. type = mac80211
					wireless .# QMP_DEVICE. macaddr =# QMP_MAC
					wireless .# QMP_DEVICE. channel =# QMP_CHANNEL
					wireless .@wifi - iface [# QMP_INDEX ]. mode = adhoc
					wireless .@wifi - iface [# QMP_INDEX ]. bssid =# QMP_BSSID
MANAGEMENT TOOLS

qmpcontrol # is located on /etc/qmp/qmp_control.sh
	Usage : /usr/bin/qmpcontrol <function > [ params ]
	Available functions :
			offer_default_gw	: Offers default gw to the network
			search_default_gw	: Search for a default gw in the network
			configure_wifi		: Configure and apply current wifi settings
			configure_network	: Configure and apply current network settings
			apply_netserver		: Start / stop netserver depending on configuration
			enable_ns_ppt		: Enable POE passtrought from NanoStation M2/5 devices . Be careful with this

qmpinfo # It uses the library provided by OpenWRT libiwinfo which gives information about WiFi devices.
	Usage : qmpinfo <question > [ options ]
	Available questions :
			modes <dev >		: supported modes by the wireless card
			channels <dev >		: supported channels and supported HT40 type (+/ -)
			ipv4			: print all ipv4 from this computer ( excluding localhost )
			hostname		: print device hostname
			bwtest <ipv6 >		: perform a bandwidth test
			nodes			: list all nodes on MESH network
			key			: print qmp key
```