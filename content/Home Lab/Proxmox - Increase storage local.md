---
title: Proxmox - Increase storage local
description:
draft: false
tags:
  - proxmox
created: 10/02/2025 18:02
updated: 23/20/2026 21:20
---
Proxmox web interface: `Datacentre -> Storage' 
Remove unwanted storages.

Resize of local:
In a Proxmox shell:
```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```


