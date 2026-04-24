---
title: Proxmox - VAAPI access on unprivileged Jellyfin LXC
description:
draft: false
tags:
  - proxmox
  - jellyfin
  - LXC
created: 10/27/2025 19:27
updated: 23/21/2026 21:21
---
Want to use my IGPU (Integrated GPU) to transcode video for my jellyfin in an unprivileged LXC container.

# Pre-requisite: 
First ensure that your **Proxmox node** have the drivers installed and loaded for you IGPU. use `vainfo` to check. or just play it safe and install them.
```
sudo apt install mesa-va-drivers  # For AMD
sudo apt install intel-media-va-driver  # For intel
```

Also you should have Jellyfin installed in an LXC and VAAPI encoding enabled in the transcoding tab of your admin dashboard. Jellyfin install script here: https://jellyfin.org/docs/general/installation/linux/
# Setup:
1: First take note of the CONTROL GROUPS (CGroups) of your IGPU render node or `renderD128` in your **Proxmox host**:
In my case it is `226:128`.
```bash
ls -lah /dev/dri
```
```output
total 0
drwxr-xr-x  3 root root        100 Jul  9 19:26 .
drwxr-xr-x 18 root root       4.3K Jul  9 19:26 ..
drwxr-xr-x  2 root root         80 Jul  9 19:26 by-path
crw-rw----  1 root video  226,   0 Jul  9 19:26 card0
crw-rw----  1 root render 226, 128 Jul  9 19:26 renderD128
```

2: Next, Allow access and mount them by appending the following in the conf file of the lxc: 
```
sudo vim /etc/pve/lxc/<CT_ID>.conf
```
```data
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
```

# 4: Next change group permission on your Proxmox machine to allow the LXC to have access to it.

4.1: First take note of you `render` group ID within you **LXC container**: 
`104` in my case.
```bash
getent group render
```
```output
render:x:104:
```

4.2: next, change the group permission of `renderD128` in your **proxmox machine** to the render group ID of the jellyfin LXC by adding `100000`:
```bash
chown :100104 /dev/dri/renderD128
```
If your wondering why is it `100104` it is because of how LXC translates user and group IDs for non-privileged LXCs. thus non-privileged LXC root is not equal to proxmoxs root. 
This is often done by binding root starting from 0 to 100000 and 65536 to 165536. meaning Root on LXC is actually the user 100000 on proxmox. so 104 on LXC is `100104` on proxmox. 
This would not be the case if you have modified or change the value of `/etc/subgid` and `/etc/subuid` which determines which user get to bind from where and by how much.  

And you should be done at this point and enjoy your Jellyfin with IGPU transcoding.


# 3: next, restart or run your jellyfin LXC.
```bash
lxc-stop 100
lxc-start 100
```

# Resource:
https://forum.proxmox.com/threads/guide-jellyfin-remote-network-shares-hw-transcoding-with-intels-qsv-unprivileged-lxc.142639/
https://kcore.org/2022/02/05/lxc-subuid-subgid/
https://pve.proxmox.com/wiki/Unprivileged_LXC_containers
https://linuxcontainers.org/lxc/manpages/man5/lxc.container.conf.5.html
