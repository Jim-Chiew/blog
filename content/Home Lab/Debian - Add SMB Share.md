---
title: Debian - Add SMB Share
description:
draft: false
tags:
created: 23/41/2025 12:41
updated: 23/01/2026 21:01
---
```bash
sudo apt update
sudo apt install cifs-utils
```

```bash
sudo mkdir -p /mnt/smbshare
```

```bash
sudo vim /root/.smbcredentials
```

```bash
username=SMBUSER
password=SMBPASSWORD
domain=WORKGROUP   # optional
```

```bash
sudo chmod 600 /root/.smbcredentials
```


```bash
sudo mount -t cifs //SERVER_IP/SHARE_NAME /mnt/smbshare \
  -o credentials=/root/.smbcredentials,iocharset=utf8,vers=3.0
```

# fstab auto mount:
```bash
sudo nano /etc/fstab
```

```bash
//SERVER_IP/SHARE_NAME  /mnt/smbshare  cifs  credentials=/root/.smbcredentials,iocharset=utf8,vers=3.0,_netdev  0  0
```

```bash
sudo mount -a
```


# Resources:
https://tecadmin.net/mounting-samba-share-on-ubuntu/
https://linuxvox.com/blog/mount-smb-share-on-linux/