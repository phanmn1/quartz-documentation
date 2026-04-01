---
tags:
  - pfsense
  - dns
  - dhcp
  - vlans
  - firewalls
  - nat
---

# Initial Install 
---
This document is to keep track of my home config of pfSense just in case I forget how to do things 

## Pre-Installation 
- Keep track of which interface is the WAN and which is the LAN

> [!info]- Personal interfaces
>  'bce0' - WAN
>  bce1' - LAN

- Have an Idea of which is the default LAN private address and CIDR range 
- Hostname 
- Primary and Secondary DNS Servers 


## Installation 
The following instructions are from Lawrence Systems Video [^1]
1. Download .iso from pfSense site
2. Run installation 
3. During Installation you will be asked to assign interfaces to WAN and LAN from the pre-installation step. 
>[!tip] WAN Interface Issues
>If you WAN interface is not getting an assigned IP from your ISP you can power cycle the modem and pfSense. 
>
> **Steps:**
> Shutoff both pfSense and Modem, Plug power back in modem **only** until all lights flash and then turn on the pfSense. The modem should be able to get the mac address of the pfSense to give it an external WAN IP. 
> 

4. At this point you should be able to see the console with 16 or so options. 
>[!tip] LAN Assignment
>Make sure you see an interface address for the LAN. Should look something like 
>**LAN -> bce1 -> v4: 192.168.1.1/24**
>That should be your gateway address to type into the URL
>
>If you don't see an address, select option 1 to assign a static IP address that will be the default network and subnet mask for you LAN. 

4. Connect to the switch and go to the IP address listed in above note. Factory default credentials for pfSense 
>username: ```admin```
>password: ```pfsense```

>[!Warning] Warning
>Change the password as soon as possible. 

## Config Wizard
After you have logged onto the pfSense for the first time you will go through a configuration wizard to set up pfSense for the first time. 

Go through the config pages:

***General Information***
Set up the following
- Hostname
- Domain
- DNS Servers
- Allow DNS Overrides or Not

***Time Server Information***
Set up following
- Timeserver
- Timezone

***WAN Interfaces***
Generally leave options alone in this page

>[!Warning] For internal homelab zones
>- Block RFC1918 Private Networks[^2] (*Uncheck*)
>- Block bogon networks[^3] (*Uncheck*)

**Configure LAN Interface**
Set up following
- LAN IP Address
- Subnet Mask

**Web GUI Password**
Reset admin password

**Reload**
Reload to set config changes 

# Personalize
---
From this section on this guide written personalization configs and where to find options as needed.

## General UI Setup
- From Menu select **System -> General Setup**
- Scroll down to **webConfigurator** and change various options
>[!Abstract] Web config options
>- Theme 
>- Dashboard Columns
>- ect...

## Change Default Port
We want to change the default administration port for the web ui. The reason is, if we ever want to run other services then there will be a port conflict on 80/443 to open up other services or open up the WAN in certain ways (HA Proxy is good example). Also it's good for some security since administration is not on the default port. 

- Go to **System -> Advanced** 
- Stay on the **Admin Access** tab 
- In the **webConfigurator** change to a different TCP port for pfSense administration 
>[!info]- Personal Port
>**TCP Port**: 10443

## SSH 
Enable SSH for secure remote administration 

- Go to **System -> Advanced** 
- Stay on the **Admin Access** tab 
- In the **Secure Shell** block select checkbox *Enable Secure Shell*
- Set the *Password or Public Key* option initially but then disable password and enable *Public Key Only* when public key is given to client


> [!todo] 
> Update guide to enable SSH access in pfSense. 
> https://www.youtube.com/watch?v=MVoe3mX_UZQ


## NAT Reflection 

- Go to **System -> Advanced**
- Select **Firewall & NAT** tab 
- Scroll to **Network Address Translation**
- Pure NAT 

## Change User Account
Create a new acct and disable default admin account for some security

1. Go to **System -> User Manager**
2. **+Add User**
3. Fill out new user, add ssh keys if necessary
4. Move group admins membership from *Not member of* to *Member of*
5. Go to **admin** account and click edit
6. *Check* This user cannot log in
7. Log out and log back in as new account

# Interface Assignments

## VLAN
Create VLAN and assign them to interfaces for network segmentation
1. Go to **Interfaces -> Assignments** 
2. Select **VLANs** tab
3. **+Add** 
4. Fill out VLAN information 
5. Set *Parent Interface* to a physical interface for tagging (I only have 1 interface which is the LAN)
6. Select **Interface Assignments** tab again
7. Click **+Add** to add interface assignment
8. Click on new interface assignment that was created (should be OPT or something)
9. Fill out assignment information 
	- Description: *[Name of interface]*
	- IPv4 Config Type: *Static* (not sure when I would need DHCP)
	- IPv6 Config type: *None*
	- IPv4 Adddress /CIDR range: *[your ip range]*
	- Uncheck *Block Private Network*, *Block Bogon networks*
10. **Save** and **Apply Changes**

The VLAN has been created but you still need to apply the firewall rules so that traffic can be forwarded. 

>[!Todo] 
>Update firewall rules only allow management VLAN to access pfSense. Right now it any VLAN to talk to any VLAN

11. Go to **Firewall -> Rules** 
12. Select name of interface created from above steps 
13. **Add** rule (down arrow one)
14. Fill out information:
	- Action: *Pass*
	- Disabled: *Uncheck*
	- Interface: *[Name of Interface]*
	- Address Family: *IPv4*
	- Protocol: *Any*
	- Source: *any*
	- Destination: *any*
	- Description: *[Describe rule]*
15. **Save**

New VLAN needs DHCP server to give IP's the belong to the network

16. Go to **Services -> DHCP Server**
17. Select interface name
18. Fill out information: 
	- Enable: *Check*
	- Range: *[Fill out IP range]*
19. **Save**

>[!todo] 
>Update doc to include part of vlan to include dns so network can get out to internet

## Alias
Aliases are reusable references that allow for us to use rules once and apply them across multiple firewall rules, NAT rules and other configurations. 

1. Go to **Firewall -> Aliases** 
2. **+Add**
3. Fill out information:
	- Name: *[Name of Alias]*
	- Description: *[Description]*
	- Type: *[Select type from dropdown]*
	- Fill out information below from dropdown type above
	- **Save**
	- **Apply Changes**














# Footnotes
[^1]: https://www.youtube.com/watch?v=fsdm5uc_LsU
[^2]: https://docs.netgate.com/pfsense/en/latest/network/addresses.html
[^3]: https://www.apnic.net/manage-ip/apnic-services/registration-services/resource-quality-assurance/what-is-a-bogon-address/
