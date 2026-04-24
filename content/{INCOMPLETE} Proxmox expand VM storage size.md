---
title:
description:
draft: true
tags:
created: 01/46/2025 22:46
updated: 23/00/2026 22:00
---
sudo fdisk /dev/sda

Press p to print the partition table.

Note down the start sector of /dev/sda3 exactly.

Press d to delete a partition, then choose 3 for /dev/sda3.

Press n to create a new partition:
    Choose primary (usually p)
    Partition number: 3
    Start sector: enter the exact same start sector you noted before
    End sector: press Enter to use the default (full available space)

Press t to set partition type:
    Enter 3 for the partition number
    Type code: if it was LVM, enter 8e (Linux LVM)
Press w to write changes and exit.

sudo partprobe /dev/sda
If that fails, reboot the system.

sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv   # if ext4
