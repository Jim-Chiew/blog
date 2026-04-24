---
title: OMV - MergerFS
description:
draft: false
tags:
  - OMV
created: 18/30/2026 17:30
updated: 23/14/2026 21:14
---
**mergerfs** is a [union filesystem](https://en.wikipedia.org/wiki/Union_mount) that makes multiple filesystems and/or directories appear as a single unified directory.
https://trapexit.github.io/mergerfs/latest/

Currently have one shared file in use and want to expend it size. The following works when one drive is in use. Note that i have mounted both drives already.

To use MergerFS on its own. view: [[MergerFS - installation]]

# installation:
to install mergerFS in OMV. you need OMV-Extra: 
https://wiki.omv-extras.org/
https://forum.openmediavault.org/index.php?thread/5549-omv-extras-org-plugin/
https://github.com/OpenMediaVault-Plugin-Developers/installScript/

```bash
sudo wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```
or
```bash
sudo curl -sSL https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

In OMV webpage: go to `System` -> `Plugins`.
Search for `mergerfs`. and isntall: `openmediavault-mergerfs`

Once install you will see it under: `Storage` -> `mergerfs`.
Click create and configure your merge folders. Here is an example:
![[OMV-MergerFS example.png]]

Now that it is done. you should see it when clicking `File Systems` when creating a `Shared Files`. 