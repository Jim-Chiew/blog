---
title: Nextcloud(snap) Change service port
description:
draft: false
tags:
  - nextcloud
created: 15/20/2025 20:20
updated: 23/11/2026 21:11
---
To change the web interface port of nextcloud.

```bash
sudo snap set nextcloud ports.http=8080
sudo snap set nextcloud ports.https=4444
sudo snap restart nextcloud

sudo snap get nextcloud ports
```