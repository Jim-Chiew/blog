---
title: Nginx Proxy Manager - Access Syncthing Docker admin portal
description:
draft: false
tags:
  - syncthing
  - nginx
  - nginx-proxy-manager
created: 16/34/2025 15:34
updated: 23/14/2026 21:14
---
# Overview:
To access the admin portal through Nginx Proxy Manager (NginxPM). 

I noticed that I cant login by just pointing my NginxPM to synchting container. so here is a guide on how to access it.

will not be setting up reverse proxy for the sync port as I published/linked the sync port to my host server. 

# Configuration:
In the Nginx Proxy Manager page, under Host -> Proxy Hosts -> \<your syncthing config\> -> Edit -> advanced:

Add the following:
```
location / {
  proxy_pass              http://<syncthing network name>:<Administrative port>/;
}
```

Example:
![[Access Docker Syncthing admin portal through Nginx Proxy Manager img 1.png]]
Default Administrative port is 8384.

The `Synching network name` is defined by how to named the network that links your synching to NginxPM. 

Example of my NginxPM docker compose:
```
services:
  nginx:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81'  # Admin port
    networks:
      - net1

networks:
  net1:
    external: true
    name: syncthing
```

Example of my Syncthing docker compose:
```
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    ports:
      - 22000:22000/tcp
      - 22000:22000/udp
    restart: unless-stopped
networks:
  default:
    name: syncthing
    external: true
```

In this case the name of syncthing network name is `syncthing`.
# Resources:
https://docs.syncthing.net/users/reverseproxy.html