---
title: Open Media Vault - Disable suspend and hibernation on lid close for laptop
description:
draft: false
tags:
  - OMV
created: 20/28/2025 09:28
updated: 23/14/2026 21:14
---
My OMV is running on an laptop that when closed goes into hibernation/sleep. 
# Disable suspend and hibernation:
Disabled at the systemd level with the following:
```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

System did not suspend upon lid close **but** logind was using a decent percentage of resources when lid is closed. Thus I changed the following to ignore lid switch:

If you just want to prevent suspending when the lid is closed you can set the following options in /etc/systemd/logind.conf:
```conf
[Login]
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore
```
Then run `systemctl restart systemd-logind.service` or reboot.  
More information is available in the manpage: `man logind.conf`

## Note:
A modern **alternative** approach for disabling suspend and hibernation is to create `/etc/systemd/sleep.conf.d/nosuspend.conf` as
```conf
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowSuspendThenHibernate=no
AllowHybridSleep=no
```
The above technique works on Debian 10 Buster and newer. See systemd-sleep.conf(5) for details.

To re-enable hibernate and suspend use the following command:
```bash
sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

# Resources:
[https://wiki.debian.org/Suspend#
Disable_suspend_and_hibernation](https://wiki.debian.org/Suspend#Disable_suspend_and_hibernation)