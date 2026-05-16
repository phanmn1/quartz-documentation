---
title:
tags:
  - freeipa
draft: true
publish: false
created: 03-30-2026
last-modified: 04-22-2026 21:04
---


# Intro
FreeIPA is an open source Identity Managment System developed by Red Hat. It provides the following services: 
- Directory Server (369 Directory Server)
- Kerberos (MIT Kerberos)
- Certificate Authority (Dogtag Certificate System)
- DNS (Bind DNS)
- SSSD 

Since I have a lot of services on my system the require authentication, I didn't want to memorize different accounts for them. I want to set a single account and have LDAP authenticate through all of them for the ideal SSO environment 

# Installation
Installation steps to get started with FreeIPA. I used Rocky Linux as it is a derivative of Red Hat and therefore FreeIPA would be more compatible with the dependency tools as well as more documentation and forums to look through if I get stuck. 

As with the documentation I have already set up a cloud-init image to deploy the VM as needed. I just need to set up the VM for FreeIPA post install. 

### Prerequisites
Have a vm of Rocky Linux ready with specs
- RAM: 4GB min (pref 8GB)
- CPU: 2vCPUs
- Storage: 10GB+
- OS: Rocky Linux 10
- Static IP


>[!todo]
>Maybe have these in ansible playbook for post rocky installation
1. Install vim/nano for text editing
```bash ln:false
sudo dnf install vim -y
```

2. Set FQDN: 
```bash ln:false
sudo hostnamectl set-hostname ipa.yourdomain.com
```

3. Set hosts file (/etc/hosts). FQDN must come first before the shortname
```config
10.22.10.2 ipa.example.com ipa 
```

4. Install Firewalld 
```bash
sudo dnf install firewalld -y
```

5. Enable Firewalld 
```bash
# Enable Firewalld
sudo systemctl enable firewalld

# Start Firewalld
sudo systemctl start firewalld
```

6. Configure ports to be open
```bash
sudo firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,dns,ntp,http,https,kerberos} --permanent
sudo firewall-cmd --reload
```

### Installation 
After pre-requistes are installed for FreeIPA, we can download the installer and run it

1. Install package 
```bash
# 1. Install the packages
sudo dnf install freeipa-server freeipa-server-dns -y

# 2. Run the clean install
sudo ipa-server-install --setup-dns --forwarder=8.8.8.8
```


NTP Servers
- ntp1.hawaii.edu
- 0.us.pool.ntp.org
- 
### Post Installation 
1. Check if (/etc/hosts) file got updated. Fix if the custom hosts entry does not correctly reflect the FQDN:
```config
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# Your Custom Logic Entry
10.x.x.x    ipa.exapmle.com ipa
```

2. Check dns resolution to make sure FreeIPA is handling it
```bash
dig @10.22.30.2 net.minhkari.com
```


> [!todo] 
> Update Mac OSX settings to override the security of untrusted certiicate authories do it doesn't redirect to public site if a public domain has been purchased

## Replica IPA
Because the Domain Controller authenticates with the server, we need to make sure that there is a backup in case the main server goes down for whatever reason. 

### Prerequisites 
The backup server needs to be a client first before it can promote to a replica server

1. Configure hosts file (see above)
2. Install Client IPA Client
	```bash
	sudo dnf install freeipa-client -y
	```

3. Join as client through ipa w/ otp (update instructions)

4. Run installer
```bash
sudo ipa-client-install --password=YOUR_OTP_HERE
```
// set chrony to pfSense NTP 

4. Promote server to replica
```bash
# 1. Install the server packages 
sudo dnf install freeipa-server freeipa-server-dns -y 

# 2. Run the replica promotion 
sudo ipa-replica-install --setup-ca --setup-dns --forwarder=8.8.8.8
```

5. Check replica status on replica and master
```bash title=On Replica
sudo ipa-replica-conncheck --master ipa01.net.minhkari.com
```

```bash title=On Master
sudo ipa-replica-conncheck --replica ipa02.net.minhkari.com
```


## LDAP Setup for others nodes to use
For other systems to sync accounts to FreeIPA, you need a bind account which can only be created in the cli. Once you have the specific bind account, you can use it to auto fetch user data from the central directory for centralized management. 

### Create Bind Account
1. Create a .ldif file to create new system account
```config title=proxmox_bind.ldif ln:false
dn: uid=proxmox-bind,cn=sysaccounts,cn=etc,dc=net,dc=minhkari,dc=com 
changetype: add 
objectclass: account 
objectclass: simplesecurityobject 
uid: proxmox-bind 
userPassword: PaoGoogl3
passwordExpirationTime: 20380119031407Z 
nsIdleTimeout: 0
```

>[!note]
>- **The Container:** By putting it in `cn=sysaccounts`, it won't show up in the FreeIPA Web UI under "Users," keeping your management clean.
>- **The Expiration:** That `2038` date is essentially "forever" in Linux time, so your Proxmox sync won't suddenly break next month.
>- **The Attributes:** Including `simplesecurityobject` is what allows it to have a standard password for the LDAP bind.

2. Import it into the Directory.
```
ldapadd -x -D "cn=Directory Manager" -W -f proxmox_bind.ldif
```

3. Check if user is added
```
```ldapsearch -x -D "cn=Directory Manager" -W -b "cn=sysaccounts,cn=etc,dc=net,dc=minhkari,dc=com" "(uid=proxmox-bind)"
```
---
### Create the Proxmox Access Group
Instead of giving every FreeIPA user access to your hypervisor, create a dedicated group.

1. Log into the **FreeIPA Web UI**.
2. Go to **Identity** > **Groups**.
3. Click **+ Add** and enter:
    - **Group name:** `pve_users` 
    - **Description:** Users allowed to log into Proxmox.
        
4. Once created, click on the `pve_users` group, go to the **User Groups** tab (or direct members), and add your own user account ie.(`mphan`) to it.
---

### Map the Attributes for Proxmox

In the Proxmox **Sync Options** tab, you need to tell it which FreeIPA fields map to Proxmox fields. Use these standard FreeIPA values:

- **User Object Class:** `person`
    
- **Group Object Class:** `groupofnames`
    
- **Unique Member Attribute:** `member`
    
- **User Attribute Name:** `uid` (This ensures you log in with your username, not your email).
    

### 4. Apply the LDAP Filter

In the Proxmox **Sync Options** window, use this filter to restrict the sync to only members of your new group:

**User Filter:** `(&(objectClass=person)(memberOf=cn=pve_users,cn=groups,cn=accounts,dc=net,dc=minhkari,dc=com))`

> **Note:** The `&` symbol is LDAP "AND" logic. It says: "Find objects that are **people** AND are **members** of the `pve_users` group."

---

### 5. Final Step: The Proxmox Permissions

Syncing the users only brings their _names_ into Proxmox. It doesn't give them permission to do anything yet.

1. In Proxmox, go to **Datacenter** > **Permissions**.
    
2. Click **Add** > **Group Permission**.
    
3. Select the synced group (e.g., `pve_users@ipa`).
    
4. Set the **Role** to `Administrator` (or whatever level you want).
    
5. Set the **Path** to `/` (for access to the whole cluster).

## create reverse zone/dns 
### 1. The Fix: Create the Reverse Zone

Before adding your nodes, you need to tell FreeIPA that it is the authority for your IP range (e.g., `10.x.x.0/24`).

1. In the FreeIPA Web UI, go to **Network Services** > **DNS** > **DNS Zones**.
    
2. Click **+ Add**.
    
3. Choose **Add reverse zone from IP**.
    
4. Enter your **Network Address** (e.g., `10.0.10.0`).
    
    - _Note: If you have a standard home network, the prefix is likely /24._
        
5. Click **Add**. FreeIPA will automatically name it something like `10.0.10.in-addr.arpa.`.
    

---

### 2. Now, Add the Records

Now that the "folder" exists, go back to your forward zone (`net.minhkari.com`) and add your node:

1. **Name:** `pve01`
    
2. **IP Address:** `10.0.10.11`
    
3. **Check the box:** "Create reverse record."
    
4. **Click Add.** It should now succeed because it has a place to put that PTR record.
    

---

### 3. Why it fails without the zone

Think of a Forward Zone as a phone book sorted by **Name**. Think of a Reverse Zone as a phone book sorted by **Number**. If you try to add a "Number" entry but the "Number" book doesn't exist, the system doesn't know where to write it down.

### 4. Manual Verification

If you already created the A record and just want to add the reverse part manually:

1. Go to your new **Reverse Zone** (`10.0.10.in-addr.arpa.`).
    
2. Click **+ Add**.
    
3. **Record name:** Use only the last number of the IP (e.g., `11`).
    
4. **Record type:** `PTR`.
    
5. **Hostname:** `pve01.net.minhkari.com.` (**Crucial:** Must end with a period).


### 
delete host from ipa 
```
kinit admin 
ipa host-del docker-host-01.net.minhkari.com
```



