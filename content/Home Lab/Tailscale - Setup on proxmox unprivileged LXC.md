---
title: Tailscale - Setup on proxmox unprivileged LXC
description:
draft: true
tags:
  - proxmox
  - LXC
  - VPN
created: 14/02/2025 16:02
updated: 23/22/2026 21:22
---
# 1. Preparation of LXC
In order LXC to work on an unprivileged LXC, we need to give it access to `/dev/tun` device.
First, use the command to get/double check the Cgroup of the device **in your proxmox host**: 
```bash
ls -l /dev/net/tun
```
```output example
user@proxmox:~$ ls -l /dev/net/tun
crw-rw-rw- 1 root root 10, 200 Jul 12 22:15 /dev/net/tun
```
in my case and in most cases the C-Group value is `10:200`

Next add/append the following to `/etc/pve/lxc/<CT_ID>.conf` config file: to give access and mount the device to the LXC:
```bash
sudo vim /etc/pve/lxc/100.conf
```
```data
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
Restart or relaunch the LXC.

# 2. Installation:
Installing it on a **Debian 12 LXC**:
For other distros, you can consult the official installation guide [here](https://tailscale.com/download/linux).
```bash
curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/debian/bookworm.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
```

```bash
sudo apt-get update
sudo apt-get install tailscale
```

# 3. Generate keys for headless authentication:
1. Go to https://login.tailscale.com/admin/settings/keys.
2. Select **Generate auth key**.
3. Add a description and generate with the default options. Feel free to change it to your needs if you want to. For my use case I do not need it to be reusable nor do I want Ephemeral.
4. Copy the key as shown before clicking okay. it only shows once. your key should look something like:
```bash
tskey-abcdef1432341818
```

# 4. Launch/Start tailscale with the authkey from above:
Use the auth keys generated in step 3:
```bash
sudo tailscale up --auth-key=tskey-abcdef1432341818
```

# 5. Configure subnet routers:
Enable ipv4_forwarding:
```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

Add Advertise network address:
```bash
sudo tailscale set --advertise-routes=192.0.2.0/24,198.51.100.0/24
```

# 6. Enable and set ACL rules in your tailscale admin portal:
Just head over to the admin console and enable the Subnet routes advertising:

Machines -> <Your tailscale machine\> -> click on the 3 dots -> Edit route setting.
Under Subnet routes, click on the network or networks that you want to be able to access and hit save.

You would also need to edit the Access controls policy under `Access controls`. You can refer to the docs to set that up.

# Resource:
https://tailscale.com/download/linux/debian-bookworm
https://tailscale.com/kb/1085/auth-keys
https://www.youtube.com/watch?v=JC63OGSzTQI
