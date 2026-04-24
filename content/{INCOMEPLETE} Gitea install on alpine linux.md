---
title:
description:
draft: true
tags:
created: 04/20/2026 16:20
updated: 23/58/2026 21:58
---
```bash
apk update
apk upgrade

apk add vim sudo
```

```bash
adduser user
adduser user wheel

id user

visudo
```
Uncomment the following:
```bash
%wheel ALL=(ALL:ALL) ALL
```

```bash
su - user
passwd -l root
sudo grep '^root:' /etc/shadow
```

```bash
apk add gitea
```

```bash
sudo chown gitea /var/lib/gitea
```

```bash
sudo chown gitea:user /etc/gitea/app.ini 
```

# 4: Edit fstab to ensure auto mounting on reboot:
Do note the UID and GIUD is the user ID and group ID of the **user you want to set in the CT/LXC**. 
So in my **jellyfin LXC**. My user and group id is:
```
id -u jellyfin  # UserID
id -g jellyfin  # GroupID
```
```
user@jellyfin:~$ id -u jellyfin
103
user@jellyfin:~$ id -g jellyfin
112
```

next add them to the value `100000`. This is due to how LXC binds user on a unprivileged user so that the root on the LXC is not root on Proxmox.

next. in your porxmox, mount it by editing the fstab file:
``` bash
sudo vim /etc/fstab
```
Add the following line:
```data
//<SMB>/<share_dir> /media/share cifs x-systemd.automount,credentials=/root/.smbcredential,uid=100103,gid=100112 0 0
```


So for my case it is:
```bash
//192.168.1.2/jellyfin /media/share cifs x-systemd.automount,credentials=/root/.smbcredential,uid=100103,gid=100112 0 0
```

# 5: Reload Fstab and mount SMB share on your **Proxmox host**:
```bash
sudo systemctl daemon-reload
sudo mount -a
```

# 6: Bind mount Proxmox share to LXC share.
Run in Proxmox: 
``` bash
pct set <CT-ID> -mp0 /media/share,mp=/share
```

Done! at this point you should see a `/share` in your LXC. if not try restarting it.



In Alpine: `vim /etc/gitea/app.ini` 

```bash
sudo -u gitea gitea
```
Visit the website to configure gitea.


```bash
rc-update add gitea
rc-service gitea start
rc-service gitea status
```

# HTTPS:
```bash
gitea cert --host [HOST]
```

`vim /etc/gitea/app.ini`
```
[server]
...
PROTOCOL  = https
CERT_FILE = /share/certs/cert.pam
KEY_FILE  = /share/certs/key.pam
REDIRECT_OTHER_PORT = true
PORT_TO_REDIRECT = 3080
```

```
sudo rc-service gitea restart
```

# Nginx:
`vim /etc/gitea/app.ini`
```
[server]
...
ROOT_URL = https://gitea.example.com/
```

```nginx
## _______________________ gitea ________________________________________
server {
    listen 80;
    server_name git.example.com;
    access_log  /var/log/nginx/host.access.log  main;
    return 301 https://git.example.com;
}

server {
    listen              443 ssl;
    server_name         git.example.com;

    access_log  /var/log/nginx/host.access.log  main;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    location / {
        client_max_body_size 512M;
        proxy_pass https://192.168.1.2:3000;
        proxy_set_header Connection $http_connection;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo systemctl reload nginx
```

# Resources: 
https://docs.gitea.com/installation/install-from-package
https://docs.gitea.com/administration/https-setup
https://docs.gitea.com/administration/reverse-proxies
https://www.cyberciti.biz/faq/how-to-enable-and-start-services-on-alpine-linux/
