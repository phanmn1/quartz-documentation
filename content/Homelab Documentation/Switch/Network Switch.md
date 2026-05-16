---
tags:
  - tplink
  - vlan
last-modified: 04-11-2026 10:04
---

# Instructions

## Static IP 
1. Access the switch’s web interface at **192.168.0.1** using a browser. 
2. Go to **L3 FEATURES** > **Static Routing** > **IPv4 Static Routing**
3. Click **Add** and fill out the following: 
	>[!Form]
	> - Destination: *0.0.0.0*
	> - Subnet Mask: *0.0.0.0*
	> - Next Hop: *[Default gateway IP]*
	> - Distance: *1 (optional)*
	click **Create**
4. Go to **L3 FEATURES** >**Interface**. 
5. In the **Interface List**, locate **VLAN 1** (default management VLAN). 
6. Click **Edit IPv4**.
7. Enter fields: 
	>[!Form] 
	>IP Address Mode: *Static* 
	>IP Address: *[static ip]*
	>Subnet Mask: *[netmask]*
	
	Click **Apply** to save.

## Add VLAN Config 
1. Go to **L2 FEATURES** > **VLAN** > **802.1Q VLAN** 
2. On **VLAN Config** tab click **+Add**
3. Fill out new VLAN:
	>[!Form]
	> - VLAN ID: *[new vlan id]*
	> - VLAN Name: *[desc name of vlan]*
	> - Untagged Ports: *[select untagged ports of vlan]*
	> - Tagged Ports: *[select tagged ports]*
	
>[!Warning] 
>**If using router on a stick:**
>Keep in mind that the pfSense port needs to have tagged ports for all vlans since it's the one doing the routing. Also keep native vlan untouched so untagged frames can also be passed through. 
>>[!Todo] 
>>Update Multilayer switch routing to make use of tp link capabilities

# LACP 
>[!todo]
>Write about Link Aggregation for synology nas and tp link jetstream setup

