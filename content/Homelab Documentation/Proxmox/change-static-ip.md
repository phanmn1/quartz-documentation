---
title: ""
description: ""
tags:
  - homelab
parent: ""
draft: true
publish: false
created: 04-10-2026
last-modified: 04-10-2026 12:04
---
>[!note] 
>This is just the basic config to get proxmox connected to the network for configs, the */etc/network/interfaces* file looks different when vlans are applied

1. Open up interfaces file 
	```console
	nano /etc/network/interfaces   
	```

2. Update config with new static ip 
	```bash
	auto vmbr0
	iface vmbr0 inet static
	address <static ip>/<cidr>/e
	gateway <gateway>
	bridge-ports eno1
	bridge-stp off
	bridge-fd 0
	dns-nameservers <dns ip> <dns ip2>
	```

3. Update the hosts file 
	```bash
	nano /etc/hosts
	```

4. Update resolve conf to have the dns be applied globally
	```bash
	nano /etc/resolv.conf
	```

5. Update config 
	```
	search <domain>
	nameserver <dns ip> 
	```

6. Restart network services
	```bash
	systemctl restart networking   
	```

7. Reboot Proxmox to apply to all config changes
	```bash
	reboot now
	```