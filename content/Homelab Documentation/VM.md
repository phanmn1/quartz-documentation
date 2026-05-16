---
title: VM Configurations
description: How to install and configure the vms for each proxmox cluster
tags:
  - homelab
  - VM
parent: ""
draft: true
publish: false
created: 05-03-2026
last-modified: 05-14-2026 11:05
---
# VMs
## QuickSync VM Passthrough 

### Pre-Requisites
- CPU w/ QuickSync
- Motherboard with IOMMU/VT-d (enable in BiIOS)
- Backup Access to Proxmox host (SSH, IPMI)

### Check hardware requirements
List the hardware components that that can be passed through. What we are looking for is "VGA compatible controller"
```console ln:false
lspci -nn
```

![[Screenshot 2026-05-04 at 8.18.01 PM.png | Result of console command]]

We are looking for a the VGA controller like: 
`00:02.0 VGA compatible controller |0300]: Intel Corporation Iris Plus Graphics 640 [8086:5926] (rev 06)`

You can see my GPU on `00:02.0`. This is the QuickSync GPU. Keep track of the ID number

### Configure Proxmox GPU Passthrough 
1. **Enable IOMMU in Proxmox**
	Edit `/etc/default/grub` file and add `GRUB_CMDLINE_LINUX_DEFAULT`

	```console 
	GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
	```

	Update grub
	```console ln:false
	update-grub
	```

2. **Add lines to VFIO modules:**  
	Edit `/etc/modules` and add lines: 
	```console
	vfio
	vfio_iommu_type1
	vfio_pci
	vfio_virqfd
	```
3. **Blacklist host GPU drivers**
	```console: ln
	echo "blacklist i915" >> /etc/modprobe.d/blacklist.conf
	```
4. **Bind GPU to VFIO**
	a. Like earlier find the GPU's PCI ID
	```console ln:false
	lspci -nn | grep -E "VGA|3D|Display"
	```
	
	You'll get an output but want to just take a look at the ID portion of the output which is something like`00:02.0`. 

	 b. Create a file `/etc/modprobe.d/vfio.conf`
	 c. Edit file and replace `GPU_ID` with your ID (ie `00:02.0`)
	 ```console
	 options vfio-pci ids=GPU_ID
	 ```
5. **Rebuild initramfs and reboot**
	```bash ln:false
	update-initramfs -u
	```
	Then reboot
	```bash ln:false
	reboot
	```





https://diymediaserver.com/post/gpu-passthrough-proxmox-quicksync-guide/