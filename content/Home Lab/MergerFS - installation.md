---
title: MergerFS - installation
description:
draft: false
tags:
  - Linux
created: 18/43/2026 16:43
updated: 23/10/2026 21:10
---
**mergerfs** is a [union filesystem](https://en.wikipedia.org/wiki/Union_mount) that makes multiple filesystems and/or directories appear as a single unified directory.
https://trapexit.github.io/mergerfs/latest/

# Installation:
Installing for `Debian GNU/Linux 12 (bookworm)`
```bash
sudo apt update
sudo apt install -y mergerfs
```
It is recommended to install using
``` bash
wget https://github.com/trapexit/mergerfs/releases/download/<ver>/mergerfs_<ver>.debian-<rel>_<arch>.deb
sudo dpkg -i mergerfs_<ver>.debian-<rel>_<arch>.deb
```
but apt is just much simpler.

use `hostnamectl` to find out your version.
visit here to find lates version of Mergerfs: https://github.com/trapexit/mergerfs/releases

for my case it is
```bash
wget https://github.com/trapexit/mergerfs/releases/download/2.41.1/mergerfs_2.41.1.debian-bookworm_amd64.deb
```
```bash
sudo dpkg -i mergerfs_2.41.1.debian-bookworm_amd64.deb
```

# Configuration: Fstab
Modifying FS will persist over reboots

to view drives:
```bash
lsblk -f
```
to mount drives: 
```bash
sudo mount <Device> <destination>
```

Configure mergeFS:
```bash
sudo vim /etc/fstab
```

```bash
/mnt/hdd0:/mnt/hdd1 /media mergerfs cache.files=off,category.create=mfs,func.getattr=newest,dropcacheonclose=false 0 0
```

In My Case:
```bash
/srv/dev-disk-by-uuid-6357ab45-3dcf-40c9-b95a-fb23ca53b042:/srv/dev-disk-by-uuid-26d85e3b-5373-429b-9d83-d591b22f3bdf /media/combined mergerfs cache.files=off,category.create=mfs,func.getattr=newest,dropcacheonclose=false 0 0
```

I changed the default behaviour for `category.create` from `pfrd` to `mfs (most free space)`. as I want the files to be spread evenly across my drives. 

here is the link to find out more:
trapexit.github.io/mergerfs/latest/config/functions_categories_policies/

Mount the device defined in fstab
```bash
sudo mount -a
```
```bash
systemctl daemon-reload
```

## To Remove:
```bash
sudo umount /media/combined
```

```bash
sudo vim /etc/fstab
```
Remove the line you added:
```bash
/srv/dev-disk-by-uuid-6357ab45-3dcf-40c9-b95a-fb23ca53b042:/srv/dev-disk-by-uuid-26d85e3b-5373-429b-9d83-d591b22f3bdf /media/combined mergerfs cache.files=off,category.create=mfs,func.getattr=newest,dropcacheonclose=false 0 0
```

```bash
sudo systemctl daemon-reload
```

## Configuration through CLI
Default quick start guide configuration: 
``` bash
mergerfs -o cache.files=off,category.create=pfrd,func.getattr=newest,dropcacheonclose=false /mnt/hdd0:/mnt/hdd1 /media
```