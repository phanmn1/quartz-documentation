---
tags: [proxmox, cloud-init]
---

## Table of Contents

1. [[#First Things to do|First Things to do]]
2. [[#Resources|Resources]]
3. [[#Create Cloud-init image|Create Cloud-init image]]
	1. [[#Create Cloud-init image#Step 1: Download Cloud Image|Step 1: Download Cloud Image]]
	2. [[#Create Cloud-init image#Step 2: Create Image Template|Step 2: Create Image Template]]
	3. [[#Create Cloud-init image#Step 3: Import Disk Image|Step 3: Import Disk Image]]


## First Things to do 
---
### 1. Disable the Enterprise Repository

You need to comment out the line that is causing the `401 Unauthorized` error.

1. Open the enterprise list file:
    
    Bash
    
    ```
    nano /etc/apt/sources.list.d/pve-enterprise.list
    ```
    
2. Add a `#` to the beginning of the line so it looks like this:
    
    Plaintext
    
    ```
    # deb https://enterprise.proxmox.com/debian/pve trixie pve-enterprise
    ```
    
3. Press `Ctrl+O`, `Enter`, then `Ctrl+X` to save and exit.
    

---

### 2. Add the No-Subscription Repository

Now you need to give Proxmox a place where it _is_ allowed to get updates.

1. Open your main sources file:
    
    Bash
    
    ```
    nano /etc/apt/sources.list
    ```
    
2. Add these lines to the bottom of the file:
    
    Plaintext
    
    ```
    # PVE no-subscription repo
    deb http://download.proxmox.com/debian/pve trixie pve-no-subscription
    ```
    
3. Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
    

---

### 3. Update and Refresh

Now that you've swapped the keys, tell Debian to check the new "No-Subscription" mirrors:

Bash

```
apt update
```

**The Result:** You should see a hit for `download.proxmox.com` and no more `401 Unauthorized` errors. You can now run `apt dist-upgrade` to get your NUCs fully up to date.

## Resources 

1. Run Updates 
	```console
	apt update && apt upgrade -y
	```


3. Backup config
Backup these files 
/etc/network/interfaces
/etc/pve

## Create Cloud-init image
Using Rocky Linux as example 

### Step 1a: Copy url and checksum from site
Steps to download image through proxmox downloader
1. Copy download url for cloud images from site ([Link](https://rockylinux.org/download))
2. Download the CHECKSUM file to make sure download was correctly received

### Step 1b: Download Image
Have Proxmox download the image from url. Rocky Linux should have a .qcow2 extension
1. From Node go to **local** -> **Import**
2. Fill out form: 
	>[!Form] 
	> - URL:  *[select **Query URL** to have proxmox find name for image]*
	> - File name: *[**Query URL** would be able to get information to fill out]*
	> - Advanced: *Checked (optional)* 
	> - Hash Algorithm:  *[CHECKSUM file for Rocky uses **SHA-256**]*
	> - Checksum: *[checksum hash from file specific to the Linux Image]*
	
	Select **Download**

### Step 2a: Create Image Template
Steps to create template image of cloud-init vm 

1. Right-click on server or go to local-lvm and click **Create VM**
2. For **General** tab 
	>[!Fill out information]
	> - Node: *[select node that template will be saved on]* 
	> - VM ID: *[Enter ID of VM]*
	> - Name: *[Enter name of template]*

	click **Next**

3. For **OS** tab select **Do not use any media** radio button and click **Next**
4. For **System** 
	>[!Fill out information]
	> - Graphic card: *Default*
	> - SCSI Controller: *VirtIO SCSI single*
	> - Machine: *q35*
	> - QEMU Agent: *Checked*
	> - Bios: *OVMF (UEFI)*
	> - Add EFI Disk: *checked*
	> - EFI Storage: *local-lvm*

	click **Next**
5. For **Disks** tab remove the assigned disk (trash Icon) and click **Next**
6. For **CPU** tab:
	>[!Fill out information]
	> - Type: *x86-64-v3/4 (if using Rocky Linux 10)*
	> - *Leave everything else default*
7. change nothing, just click **Next**
8. For **Memory** tab 
	>[!Fill out information] 
	> - Memory (MiB): *[set memory to whatever, can change later. 1024 is usually appropriate]*
	
	click **Next**
9. For **Network** tab change nothing and click **Next**
10. For **Confirm** tab review settings 
>[!Danger] Make sure that **Start after created** is NOT CHECKED!! 
>![[Screenshot 2026-01-27 at 1.51.30 PM.png]] 

	click **Next**


### Step 2b: Configure newly created template
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


### Step 3: Import Disk Image
Import .qcow2 image into a useable boot device for cloud-init to be able to use 

1. SSH into proxmox or use 
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
	>[!Form]
	> - Discard: *[If on SSD then check, if not then leave alone]*
	> - SSD emulation: *[If SSD then check, if not then leave alone*]
	> - *All other settings leave alone*
	
	Click **Add**

### Step 4: Edit boot order
We are done with the hardware configs. Lets edit the boot order 

1. From node click on VM and go to **Options** tab
2. Select **Boot Order** and click **Edit**
3. Disable network and enable disk
4. Click **Ok**

### Step 5: Convert to template
1. Right click on VM and select **Convert to Template**

### Step 6: Configure Cloud-init 
>[!Todo]
>Write out form to fill out cloud init stuff


## Change static ip through CLI
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

5. Reboot Proxmox to apply to all config changes
	```bash
	reboot now
	```

## OpenTofu
### Initialize OpenTofu 
1. **Create the Role**:
    - Go to **Datacenter** > **Permissions** > **Roles**.
    - Click **Create**.
    - Name the role `TerraformProv`.    
    - Select all the required privileges from the list (e.g., `VM.Allocate`, `VM.Clone`, `Datastore.AllocateSpace`, etc.).      
    - Click **Create**.
2. **Create the User**:
    - Go to **Datacenter** > **Permissions** > **Users**.
    - Click **Add** > **User**.
    - Set **User Name** to `terraform-prov`.
    - Set **Realm** to `Proxmox VE authentication server (pve)`.
    - Set a password.
    - Click **Create**.
3. **Assign the Role to the User**:
    - Go to **Datacenter** > **Permissions**.
    - Click **Add** > **User Permission**.
    - Set **Path** to `/`.
    - Select **User** as `terraform-prov@pve`.
    - Select **Role** as `TerraformProv`.
    - Ensure **Propagate** is checked.
    - Click **Add**.
4. **Create the API Token**:
    - Go to **Datacenter** > **Permissions** > **API Tokens**.
    - Click **Add**.
    - Select **User** as `terraform-prov@pve`.
    - Set **Token ID** to `tofu`.
    - **Uncheck "Privilege Separation"**.
    - Click **Add**.
    - **Immediately copy the generated Token Secret (value)**.  It will not be shown again.

## Add to cluster
## Gemini said
Step 1: Initialize the Cluster

1. Log into the Web UI of your first NUC.
    
2. Navigate to **Datacenter** (at the very top of the left tree).
    
3. Select **Cluster** from the middle menu.
    
4. Click the **Create Cluster** button.
    

### Step 2: The Logic Configuration

A dialog box will appear with these key fields:

- **Cluster Name:** Give it a clean, descriptive name (e.g., `Minhkari-Cluster` or `NUC-Lab`). _Note: No spaces allowed here._
    
- **Cluster Network:** This is the most important part for your "Great Migration." Look for the dropdown menu labeled **Link 0**.
    
    - **Select your 10.x.x.x IP address.** * **The Logic:** You want the high-speed, low-latency traffic of the cluster communication to stay on your internal LAN, not routing out through a gateway or a slower interface.
        
- **Link 1 (Optional):** If your NUCs have a second ethernet port or a Thunderbolt-to-Ethernet adapter, you can add a second link for redundancy. If one cable fails, the cluster stays alive.
    

---

### Step 3: Execution

Click **Create**. You’ll see a task log window pop up. It will generate a new set of encryption keys and restart the `pve-cluster` and `corosync` services.

> **Note:** Your Web UI might "flicker" or show a connection error for 5–10 seconds. This is normal as the cluster filesystem (`pmxcfs`) mounts for the first time.

### Step 4: Verification

Once the task says `TASK OK`:

1. The **Cluster** tab should now show your first node listed with **1 Vote**.
    
2. The **Join Information** button (which was greyed out before) should now be clickable.

### Step 2: The Handshake (On Node 2)

1. Log into the Web UI of your **newly installed** NUC.
    
2. Go to **Datacenter > Cluster**.
    
3. Click **Join Cluster**.
    
4. Paste the long string of text you just copied into the **Information** box.
    
5. The IP and Fingerprint fields should auto-fill.
    
6. **Password:** Enter the `root` password for **Node 1**.
    
7. **Cluster Network:** Ensure the dropdown is set to your **10.x.x.x** management interface.
    
8. Click **Join Cluster**.


# Footnotes
---
https://technotim.live/posts/first-11-things-proxmox/
https://www.youtube.com/watch?v=1Ec0Vg5be4s