---
title: Cloud-init (Rocky Linux)
description: ""
tags:
  - homelab
  - cloud-init
parent: "[[proxmox-main]]"
draft: false
publish: true
created: 04-10-2026
last-modified: 04-11-2026 09:04
---
# Cloud Init - Rocky Linux 


## Step 1a: Copy url and checksum from site
Steps to download image through proxmox downloader
1. Copy download url for cloud images from site ([Link](https://rockylinux.org/download))
2. Download the CHECKSUM file to make sure download was correctly received
---

## Step 1b: Download Image
Have Proxmox download the image from url. Rocky Linux should have a .qcow2 extension
1. From Node go to **local** -> **Import**
2. Fill out form: 
	- **URL**:  [select **Query URL** to have proxmox find name for image]
	- **File name**: *[**Query URL** would be able to get information to fill out]*
	- **Advanced**: Checked (optional)
	- **Hash Algorithm**:  *[CHECKSUM file for Rocky uses **SHA-256**]*
	- **Checksum**: *[checksum hash from file specific to the Linux Image]*
	
	Select **Download**
---

## Step 2a: Create Image Template
Steps to create template image of cloud-init vm 

1. Right-click on server or go to local-lvm and click **Create VM**
2. For **General** tab 
	- **Node**: [select node that template will be saved on]
	- **VM ID**: [Enter ID of VM]
	- **Name**: [Enter name of template]
	
	click **Next**

3. For **OS** tab select **Do not use any media** radio button and click **Next**
4. For **System** 
	- **Graphic card**: `Default`
	- **SCSI Controller**: `VirtIO SCSI single`
	- **Machine**: `q35`
	- **QEMU Agent**: Checked
	- **Bios**: `OVMF (UEFI)`
	- **Add EFI Disk**: Checked
	- **EFI Storage**: `ocal-lvm`

	click **Next**
5. For **Disks** tab remove the assigned disk (trash Icon) and click **Next**
6. For **CPU** tab:
	- **Type**: `x86-64-v3`or `x86-64-v4` (if using Rocky Linux 10)*
	- *Leave everything else default*
7. For **Memory** tab:
	- **Memory (MiB)**: `1024`
	*Can set whatever but 1024 is okay for now*
	
	click **Next**
8. For **Network** tab change nothing and click **Next**
9. For **Confirm** tab review settings 
>[!Danger] Make sure that **Start after created** is NOT CHECKED!! 
>![[Screenshot 2026-01-27 at 1.51.30 PM.png]] 

	click **Next**

---

## Step 2b: Configure newly created template
1. Go to newly created VM template and remove CD/DVD Drive
![[Screenshot 2026-01-27 at 2.01.32 PM.png]]
2. Go to **Add** -> **CloudInit Drive** 
	>[!Fill out]
	>- Bus/Device: IDE,
	>- Storage: *local-lvm* 
	
	click **Add**
3. Go to **Add** -> **Serial Port** and set serial port number (eg. 0)
4. Click **Display** then **Edit** and set the following: 
	>[!Form]
	> - Graphic card: *Serial Terminal [serial value]*
	
	click **OK**
---


## Step 3: Import Disk Image
Import .qcow2 image into a useable boot device for cloud-init to be able to use 

1. SSH into proxmox or use the terminal in the UI
2. Import in vm
	```console 
	qm importdisk <VMID> /path/to/rocky-linux.qcow2 <StorageID>
	```
	>[!note]
	>My proxmox config the .qcow2 image was saved here. My storageid: local-lvm
	>/var/lib/vz/import
3. Go to your node and click on the VM that you extracted the image to. In the **Hardware** tab, you should see a new **Unused Disk**

	![[Screenshot 2026-03-21 at 8.53.39 PM.png]]
4. Select the disk and click **Edit**
5. Fill out form: 
	
	- **Discard**: Checked (If on SSD then check, if not then leave alone)
	- **SSD emulation**: Checked (If SSD then check, if not then leave alone)
	- *All other settings leave alone*
	
	Click **Add**

## Step 4: Edit boot order
We are done with the hardware configs. Lets edit the boot order 

1. From node click on VM and go to **Options** tab
2. Select **Boot Order** and click **Edit**
3. Disable network and enable disk
4. Click **Ok**
---

## Step 5: Convert to template
1. Right click on VM and select **Convert to Template**
---
## Step 6: Configure Cloud-init 
>[!Todo]
>Write out form to fill out cloud init stuff