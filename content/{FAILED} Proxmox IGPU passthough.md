---
title:
description:
draft: true
tags:
created: 09/35/2025 13:35
updated: 23/59/2026 20:59
---
#failed

Integrated GPU are GPU within the CPU. they are excellent for video encoding and decoding. 

## Verifying IOMMU
Your system need to ensure it supports Input-output memory management unit(IOMMU). this means CPU, IGPU, motherboard needs to have that capability. 

verify with:
```bash
dmesg | grep -e DMAR -e IOMMU  # Verify IOMMU is enabled
```

you should see output that states something like enabled supported or detected. if no output means something wrong.

Next check IOMMU interrupt remapping
```bash
dmesg | grep 'remapping'  # Verify IOMMU interrupt remapping is enabled
```
you should see `Interrupt remapping enabled`.  if not refer to the proxmox doc for instruction [here](https://pve.proxmox.com/wiki/PCI_Passthrough#Requirements)

## Configuring Grub to enable IOMMU
Switch to root if you are not:
```bash
sudo -i
```

edit grub
```bash
vim /etc/default/grub
```

Modified the following:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt pcie_acs_override=downstream,multifunction video=efifb:off"

# For intel
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

Update grub:
```
# Turn iommu on
update-grub
```

## Enabling IOMMU interrupt remapping
```bash
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```

## Blacklisting Drivers
disallow host to load drivers.
```bash
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```
Note that after this command you will not be able to use the IGPU if that is the only GPU you have. because the driver is blocked. meaning no more HDMI/video output meaning no more HDMI/video output after `update-initramfs -u` command and  reset.
## Adding GPU to VFIO
```bash
lspci -v
```
Using the above command list devices. find you GPU. something like `VGA compatible controller`.
take note of the number like `04:00.0`.

```bash
lspci -n -s 04:00
```
Take note of then vendor ID code:
example:
```
root@proxmox:~# lspci -n -s 04:00
04:00.0 0300: 1002:1638 (rev d1)
04:00.1 0403: 1002:1637
04:00.2 1080: 1022:15df
04:00.3 0c03: 1022:1639
04:00.4 0c03: 1022:1639
04:00.5 0480: 1022:15e2 (rev 01)
04:00.6 0403: 1022:15e3
```

would be:
```
1002:1638
1022:15e2
```

Next run the following command with then vendor ID.
```bash
echo "options vfio-pci ids=<ID-1>,<ID-2> disable_vga=1"> /etc/modprobe.d/vfio.conf
```
so for my case
```bash
echo "options vfio-pci ids=1002:1638,1022:15e2 disable_vga=1"> /etc/modprobe.d/vfio.conf

echo "options vfio-pci ids=1002:1638,1022:15e2,1002:1637 disable_vga=1"> /etc/modprobe.d/vfio.conf
```
Note that after the following command you will not be able to use the IGPU if that is the only GPU you have. meaning no more HDMI/video output after reset!

Update:
```
update-initramfs -u
```

reboot:
```bash
reboot
# reset # If using dedicated external GPU
```

Ensure device is being used by vfio-pci
```
 lspci -vvv -s 04:00.0
```

```
Kernel driver in use: vfio-pci
```

## assign to VM
```
sudo qm set 503 -hostpci0 04:00.0,pcie=on,x-vga=off
sudo qm set 503 -hostpci1 04:00.1,pcie=on,x-vga=off
sudo qm set 503 -hostpci2 04:00.5,pcie=on,x-vga=off

# sudo qm set 503 -delete hostpci0 # To remove
```

# Resources:
https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/
https://<Proxmox host IP address\>/pve-docs/chapter-qm.html#_general_requirements
https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)
https://3os.org/infrastructure/proxmox/gpu-passthrough/igpu-split-passthrough/
