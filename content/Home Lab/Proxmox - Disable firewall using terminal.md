---
title: Proxmox - Disable firewall using terminal
description:
draft: false
tags:
  - proxmox
  - firewall
created: 25/52/2025 07:52
updated: 23/19/2026 21:19
---
When you lock your self out of the web GUI. here is how to disable firewall in the terminal.

```bash
sudo vim /etc/pve/firewall/cluster.fw
```

Change the value of `enable` from `1` to `0`.