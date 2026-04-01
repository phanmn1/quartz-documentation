---
tags: [freeipa]
---

# Intro

# Installation
### Prerequisites
Have a vm of Rocky Linux ready with specs
- RAM: 4GB min (pref 8GB)
- CPU: 2vCPUs
- Storage: 10GB+
- OS: Rocky Linux 10
- Static IP

Install vim/nano for text editing
Maybe have this as ansible script for post rocky installation
```bash
sudo dnf install vim -y
```

### Installation 
1. Set FQDN: 
```bash
sudo hostnamectl set-hostname ipa.yourdomain.com
```

2. Set hosts file (/etc/hosts). FQDN must come first before the shortname
```bash
10.22.10.2 ipa.example.com ipa 
```

3. Install Firewalld 
```bash
sudo dnf install firewalld -y
```

4. Configure ports to be open
```
sudo firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,dns,ntp,http,https,kerberos} --permanent
sudo firewall-cmd --reload
```

5. Install package 
```bash
# 1. Install the packages
sudo dnf install freeipa-server freeipa-server-dns -y

# 2. Run the clean install
sudo ipa-server-install --setup-dns --forwarder=8.8.8.8
```

### Post Installation 
1. Keep list 
/etc/hosts 
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# Your Custom Logic Entry
10.x.x.x    ipa.exapmle.com ipa
```

2. Check the search domain 
```
search net.minhkari.com
nameserver 10.x.x.x (or your current working DNS)
```

