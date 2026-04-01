---
tags: [switchports, configs]
---

**Table of Contents**

1. [[#Networks|Networks]]
	1. [[#Networks#IP Address Ranges|IP Address Ranges]]
	2. [[#Networks#VLAN|VLAN]]

## Networks 
---
The inner working design of my home networking design. 
### IP Address Ranges
Considering the multiple devices on this network along with all the applications that I'll spin up, I'll need to plan out the network correctly. I'll need both dynamic and static IP addresses to assign to the physical devices on the network along with VM's/Containers that are bridged into the homelab network for easier networking. 

| **Static** | **Dynamic**|  
| -----------| -------------|
| **Physical ** | **VM**| 

All Calculations are 
Site Used to generate random IP : [Random IP Generator](https://www.ipvoid.com/random-ip/)
Site Used to generate cidr range : [CIDR Range Calculator](https://cidr.xyz/)
10.22.10.0/27 -> 32 (Static IP Physical Addresses)
10.22.20.0/23 -> 512 (Dynamic IP Physical Addresses)
10.22.30.0/27 -> 32 (Static IP (VM Addresses))
10.22.40.0/23 -> 512 (Dynamic IP (VM Addresses))

### VLAN 
Each Network address will have its own Vlan 
VLAN 1 - 10.22.<font color="#cc241d">10</font>.0/27 
VLAN 2 - 10.22.<font color="#cc241d">20</font>.0/23
VLAN 3 - 10.22.<font color="#cc241d">30</font>.0/27
VLAN 4 - 10.22.<font color="#cc241d">40</font>.0/23

## Static IP Physical Addresses
| **Machine** | **IPv4 Address** | **VLAN** | **Desc**
| :---------:| :-----------: | :-----: | :---: |
| pfSense | 10.22.10.1 | untagged (VLAN1) | LAN
| pfSense | 10.22.20.1 | 20 | Home 
| pfSense | 10.22.30.1 | 30 | Static VM
| Switch |  10.22.10.9 |  untagged (VLAN1) | LAN
| Sakura | 10.22.10.2 |  untagged (VLAN1) | LAN
| Haru | 10.22.10.3 | untagged (VLAN1) | LAN
| Azuki | 10.22.10.4 | untagged (VLAN1) | LAN



## Static IP VM Addresses
| **Machine** | **IPv4 Address** | 
| :---------:| :-----------: |
| IPA-DC-1 | 10.22.30.2 | 
| IPA-DC-2 | 10.22.30.3 | 
| Plex | | 


# Switch Ports 

## Table 
|**Port** | **Machine**|
| :-------:| :------:|
| 1 | Haru |
| 2 | Sakura | 
| 3 | Lady | 
| 4 | Dimitri | 
| 5 | Azuki |
| 6 | Leo |
| 7 | **NONE**
| 8 | Ivanki Hub | 
| 9 | Router (pfSense) | 
| 10| TrueNas (LACP1) |
| 11| TrueNas (LACP2) |
| 12| Synology (LACP1)|
| 13| Synology (LACP2)| 
| 14| TrueNas (IPMI) |
| 15| Hue Syncbox | 
| 16| **NONE** |
| 17| Complex (LACP1) |
| 18| Complex (LACP2) |
| 19| PC |
| 20| **NONE** |
| 21| **NONE** |
| 22| Orbi AP | 
| 23| Pikvm |
| 24 |**NONE** | 
 







