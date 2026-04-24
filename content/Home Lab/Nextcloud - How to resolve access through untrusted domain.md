---
title: Nextcloud - How to resolve access through untrusted domain
description:
draft: false
tags:
  - nextcloud
created: 15/17/2025 20:17
updated: 23/11/2026 21:11
---
Nextcloud needs to specify a trusted domain else client are not able to connect.

/var/snap/nextcloud/current/nextcloud/config/config.php
```config
'trusted_domains' =>
  array (
    0 => '192.168.1.1',
  ),
```