---
title:
description:
draft: true
tags:
created: 17/17/2025 22:17
updated: 23/00/2026 22:00
---
#nginx #ssl/tls #cryptography 
# Overview of signing:
![[Nginx - Mutual TLS setup img 1.png]]


```bash
openssl req -new -newkey rsa:2048 -nodes \
  -keyout server.key -out server.csr \
  -subj "/CN=your.server.local" \
  -addext "keyUsage=digitalSignature,keyEncipherment" \
  -addext "extendedKeyUsage=serverAuth"
```

```bash
openssl req -new -newkey rsa:2048 -nodes \
  -keyout client.key -out client.csr \
  -subj "/CN=clientuser" \
  -addext "keyUsage=digitalSignature" \
  -addext "extendedKeyUsage=clientAuth"
```
# Resource:
https://blog.devops.dev/handshake-at-first-sight-setting-up-mtls-with-nginx-3b306ec7061d
