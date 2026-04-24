---
title: Proxmox - OpenMediaVault VM SMB share to unprivilege Jellyfin LXC container
description:
draft: false
tags:
  - OMV
  - proxmox
  - jellyfin
  - LXC
created: 11/16/2025 16:16
updated: 23/20/2026 21:20
---
Quick summary: Mount SMB to Proxmox host then mount it to Jellyfin LXC. Proxmox is acting as a middleware.

Unable to mount it direcetly to LXC due to an Unprivilege LXC not having access to fstab.

# Setup:
1: Ensure you have the following package is installed **in Proxmox**:
``` bash
sudo apt install cifs-utils 
```

2: Make a mount directory:
``` bash
sudo mkdir /media/share 
```

3: Create a credential file:
```bash
sudo vim /root/.smbcredential
```
```data
username=username
password=password
```

3.2: change permissions on the file to only root having read access:
```bash
sudo chmod 400 /root/.smbcredential
```

# 4: Edit fstab to ensure auto mounting on reboot:
Do note the UID and GIUD is the user ID and group ID of the **user you want to set in the CT/LXC**. 
So in my **jellyfin LXC**. My user and group id is:
```
id -u jellyfin  # UserID
id -g jellyfin  # GroupID
```
```
user@jellyfin:~$ id -u jellyfin
103
user@jellyfin:~$ id -g jellyfin
112
```

next add them to the value `100000`. This is due to how LXC binds user on a unprivileged user so that the root on the LXC is not root on Proxmox.

next. in your porxmox, mount it by editing the fstab file:
``` bash
sudo vim /etc/fstab
```
Add the following line:
```data
//<SMB>/<share_dir> /media/share cifs x-systemd.automount,credentials=/root/.smbcredential,uid=100103,gid=100112 0 0
```


So for my case it is:
```bash
//192.168.1.2/jellyfin /media/share cifs x-systemd.automount,credentials=/root/.smbcredential,uid=100103,gid=100112 0 0
```

# 5: Reload Fstab and mount SMB share on your **Proxmox host**:
```bash
sudo systemctl daemon-reload
sudo mount -a
```

# 6: Bind mount Proxmox share to LXC share.
Run in Proxmox: 
``` bash
pct set <CT-ID> -mp0 /media/share,mp=/share
```

Done! at this point you should see a `/share` in your LXC. if not try restarting it.

# Resource: 
https://go2engle.com/posts/UnprivilegedLXCContainerSMBShare/
https://tecadmin.net/mounting-samba-share-on-ubuntu/
https://www.youtube.com/watch?v=aEzo_u6SJsk