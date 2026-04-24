---
title: Proxmox - Hard Drive Passthrough to VM.
description:
draft: false
tags:
  - proxmox
  - VM
created: 11/14/2025 21:14
updated: 23/20/2026 21:20
---
#proxmox #VM 
# Setup: 
List available devices by id: 
```bash
ls -l /dev/disk/by-id/
```

Find the drive you want to pass through to your VM. Ensure you choose it by id as names like `sda` can change. NOTE: to pass the entire drive thus use `sda` instead of `sda1`

Next, enter the following command to mount it to the virtual machine by its `VM_id`. 
```
sudo qm set <VM_ID> -scsi1 /dev/disk/by-id/<ID_HERE>,backup=0
```
Note: I disabled backups for this drive with `,backup=0` as I did not want it to be added to snapshots. 
Note2: If you are passing through multiple increment `-scsi1`. so additional one will be `-scsi2`.

# Resource:
https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)