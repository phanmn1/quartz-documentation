---
title: Proxmox
tags:
  - proxmox
  - cloud-init
parent: "[[index]]"
draft: false
publish: true
created: 01-27-2026
last-modified: 05-14-2026 11:05
weight: 1
---
# Proxmox 
## Post Installation
After installation you will want to make sure to disable enterprise repository and enable community repos 

### Update Repository Configs 
#### Method 1: GUI
This is the recommended method for most users as it handles repository configuration safely through the interface.

1. Navigate to **Datacenter** > **Your Node** > **Updates** > **Repositories**. 
2. Select the **pve-enterprise** repository and click **Disable**.     
3. Select the **ceph** repository (if present) and click **Disable**. 
4. Click **Add**, select **No-Subscription** from the dropdown menu, and click **Add** (ignore any warning about missing subscriptions). 
5. Go to **Updates** and click **Refresh** to pull the latest package list.
---

### Update packages
1. Backup configs 
	- `/etc/network/interfaces`
	- `/etc/pve`

2. Prepares system to identify new packages, then install them
	```console ln:false
	apt update && apt upgrade -y
	```
---

## Create Cloud-init images

- [[cloud-init-rocky | Cloud Init (Rocky Linux)]]
- [[cloud-init-ubuntu| Cloud Init (Ubuntu)]] 





## Change Static IP
- [[change-static-ip| Change Static IP - CLI]]

## Set network to be vlan aware 
>[!todo]
>Create instructions for vlan aware proxmox config
## OpenTofu Configs
To be able to use IaC for auto provisioning of VM's, we need to create a user and give it roles to act on behalf of OpenTofu. Then create a token to give to OpenTofu to use to spin up vm's on demand. 

### Initialize OpenTofu 
1. **Create the Role**:
    - Go to **Datacenter** > **Permissions** > **Roles**.
    - Click **Create**.
    - Name the role `TerraformProv`.    
    - Select all the required privileges from the list:   
	    - Datastore.AllocateSpace 
	    - Datastore.AllocateTemplate 
	    - Datastore.Audit 
	    - Pool.Allocate 
	    - Sys.Audit 
	    - Sys.Console 
	    - Sys.Modify 
	    - VM.Allocate 
	    - VM.Audit 
	    - VM.Clone 
	    - VM.Config.CDROM 
	    - VM.Config.Cloudinit 
	    - VM.Config.CPU 
	    - VM.Config.Disk 
	    - VM.Config.HWType 
	    - VM.Config.Memory 
	    - VM.Config.Network 
	    - VM.Config.Options 
	    - VM.GuestAgent.Audit
	    - VM.GuestAgent.Unrestricted
	    - VM.Migrate 
	    - VM.Monitor -- NOT on proxmox 9??
	    - VM.PowerMgmt 
	    - SDN.Use
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

## Remove node from one cluster to another
**Phase 1: Preparation (The "Clean Break")**  
Warning: You must move or backup all VMs/Containers on the node before starting. A node cannot be moved while it is hosting guests that belong to the old cluster's resource pool.  

1. **Evacuate the Node:** Migrate all VMs to other nodes in the old cluster.  
    
2. **Delete the Node from the Old Cluster:** On a _different_ node in the old cluster, run:  
    

```bash
pvecm delnode <nodename>
```

  

**Phase 2: Removing the Old Cluster Identity**  
Log in to the node you are moving via SSH. You need to strip its "cluster memory" so it thinks it is a standalone server again.  

1. **Stop the Cluster Services:**  
    

```bash
systemctl stop pve-cluster corosync
```

  
2. **Force Local Mode:**  
    Start the filesystem in local mode to allow editing of protected files:  
    

```bash
pmxcfs -l
```

  
2. **Delete Cluster Config Files:**  
    

```bash
rm /etc/pve/corosync.confrm -rf /etc/corosync/*
```

  
2. **Restart Services:**  
    

```bash
killall pmxcfssystemctl start pve-cluster
```

  

At this point, the node's GUI should show it as a standalone node, but it might still "see" the old nodes in the `/etc/pve/nodes` directory. You can manually delete those old folders to clean up the sidebar.  
**Phase 3: Joining the New Cluster**  
Now that the node is "single," you can join it to your new cluster (e.g., moving a NUC from your first cluster to the Geekom cluster).  

1. **Via GUI:**  
    • Go to the **New Cluster** -> **Datacenter** -> **Cluster** -> **Join Information**.  
    • Copy the Information string.  
    • Go to the **Moving Node** -> **Datacenter** -> **Cluster** -> **Join Cluster** and paste the string.  
    
2. **Via CLI (Recommended for speed):**  
    On the node you are moving, run:  
    

```bash
pvecm add <IP-of-new-cluster-node>
```

**The Cleanup Procedure**  
Run these commands from the shell of any active node in your current cluster:  

1. **List the directories** to identify exactly which ones are "ghosts":  
    

```bash
ls -l /etc/pve/nodes
```

  
2. **Delete the old node directory:**  
    Replace `<old-node-name>` with the name of the NUC or Geekom unit you just moved.  
    

```bash
rm -rf /etc/pve/nodes/<old-node-name>
```

## Resources 
# Footnotes
---
https://technotim.live/posts/first-11-things-proxmox/
https://www.youtube.com/watch?v=1Ec0Vg5be4s