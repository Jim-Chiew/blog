---
title: Proxmox - Enable memory ballooning for VM
description:
draft: false
tags:
  - proxmox
  - VM
created: 12/48/2025 22:48
updated: 23/20/2026 21:20
---
Dynamically allocate memory to your VM based on their usage. 

As it turns on you can't just check balloon in the selection menu and have it work right out of the box. some configuration is needed. 

# Method 1: QEMU Guest Agent 
For an easiest way to set is up is to have qemu guest agent running. setup of it. Using the webUI within the guest VM page: Options -> QEMU Guest Agent -> Enable.

On a linux VM, enter:
```bash
sudo apt install qemu-guest-agent
```
```bash
sudo systemctl enable qemu-guest-agent
sudo systemctl start qemu-guest-agent
```

On windows, there will be a drive that will be mounted. open it and find the executable to install.


# Method 2: Manual Configure
You need the virtio_balloon driver running for it to work. To check, enter the following in the VM: 
```bash
lsmod | grep virtio_balloon
``` 
You should see something like:
```bash
user@aRandomVM:~$ lsmod | grep virtio_balloon
virtio_balloon         12345  0
virtio                 23456  4 virtio_balloon,virtio_scsi,virtio_pci,virtio_net
virtio_ring            34567  4 virtio_balloon,virtio_scsi,virtio_pci,virtio_net
```

If it did not output anything load it manually with:
```bash
sudo modprobe virtio_balloon
```
finally, add it to `/etc/modules` to load the module at boot:
```bash
sudo vim /etc/modules
```
```data
virtio_balloon
```

Of course be sure to enable it **in Proxmox node** either through the web UI or with the shell:
```bash
qm set <VMID> --balloon <size>
```


# Resource:
https://forum.proxmox.com/threads/linux-guests-virtio_balloon-driver.139969/
https://www.youtube.com/watch?v=wp4kCUM6dik