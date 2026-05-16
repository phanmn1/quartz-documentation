---
title: Cloud-Init (Ubuntu)
description: ""
tags:
  - homelab
parent: "[[proxmox-main]]"
draft: true
publish: false
created: 04-10-2026
last-modified: 04-10-2026 17:04
---

# Cloud Init (Ubuntu)

>[!todo]
>Update Documentation (slightly modified with these instructions in the video)
>https://www.youtube.com/watch?v=1Ec0Vg5be4s


location of iso images on proxmox
note:
```
/var/lib/vz/template/iso
```

`apt install libguestfs-tools -y`

`virt-customize -a ubuntu-noble-cloudimg.img --install qemu-guest-agent`

