---
title: Nginx - Installation and Setup a Reverse Proxy
description:
draft: false
tags:
  - nginx
  - networking
  - web
  - ssl/tls
created: 17/56/2025 09:56
updated: 23/13/2026 21:13
---
Installing nginx in Debian with certbot to generate a wildcard SSL/TLS cert for a cloudflare DNS domain.
# Pre-requisite: 
In cloudflare, you should already have a record pointing the domain name to the devices you are installing nginx on. this guide does not cover that.

# Installing certbot
```bash
sudo apt install certbot python3-certbot-dns-cloudflare
```

# Generate Cloudflare API key:
For certbot to generate a certificate. authentication is needed to ensure that you own the particular domain you are generating a certificate for. For this an API key is needed. 

First visit https://dash.cloudflare.com.
Go to: Manage Account -> Account API Tokens -> Create token:
![[nginx - installation and setup a reverse proxy img 1.png]]

Use `edit zone DNS` template.
![[nginx - installation and setup a reverse proxy img 2.png]]

Under `Zone resources` choose the domain you want to generate a cert for: 
![[nginx - installation and setup a reverse proxy img 3.png]]

Lastly click on `Create Token`.

# Create cloudflare config init file:
Create and add the API key in the init file:
```bash
sudo vim /root/cloudflare.ini
```
```data
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567
```

Add the appropriate permissions: 
```bash
sudo chmod 400 /root/cloudflare.ini
```

# Generate the cert:
```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/cloudflare.ini \
  -d *.example.com
```
change the domain to your domain.
fill in the command promt and the cert should be generated in `/etc/letsencrypt/live/example.com`

Verifying Certbot Auto-Renewal:
checking demon:
```bash
sudo systemctl status certbot.timer
```

dry run:
```bash
sudo certbot renew --dry-run
```
you should see: `Congratulations, all simulated renewals succeeded: `

# Installation of nginx: 
Install the prerequisites:
```bash
sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring
```

Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:
```bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
| sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```        

Verify that the downloaded file contains the proper key:
```bash
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

**The output** should contain the full fingerprint 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 as follows:
```output
pub   rsa2048 2011-08-19 [SC] [expires: 2027-05-24]
   573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```
Note that the output can contain other keys used to sign the packages.

To set up the apt repository for stable nginx packages, run the following command:
```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
 http://nginx.org/packages/debian `lsb_release -cs` nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list
```

If you would like to use mainline nginx packages, run the following command instead:
```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
 http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list
```

Set up repository pinning to prefer our packages over distribution-provided ones:
```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
| sudo tee /etc/apt/preferences.d/99nginx
```

To install nginx, run the following commands:
```bash
sudo apt update
sudo apt install nginx
```

Start the service  with:
```bash
sudo systemctl start nginx
```

# nginx reverse proxy configuration:
For NGINX Open Source, the location depends on the package system used to install NGINX and the operating system. It is typically one of `/usr/local/nginx/conf`, `/etc/nginx`, or `/usr/local/etc/nginx`.

in my case it is `/etc/nginx`.

Main config file is: `/etc/nginx/nginx.conf` which defines log files and format and the config files to load.
all web config proxy redirects is defined in the path `/etc/nginx/conf.d/*.conf`.

If you are lazy like me you can just append the config under `/etc/nginx/conf.d/default.conf`
Below is an example for jellyfin:
```conf
server {
    listen 80;
    listen [::]:80;
    server_name jellyfin.example.com;
    return 301 https://jellyfin.example.com;
}

server {
    listen              443 ssl;
    server_name         jellyfin.example.com;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    #ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    client_max_body_size 20M;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_pass  http://192.168.1.2:8096;
    }

    location /socket {
        # Proxy Jellyfin Websockets traffic
        proxy_pass http://192.168.1.2:8096;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
    }
}
```

restart/Reload the service with:
```bash
sudo systemctl reload nginx
```
```bash
sudo systemctl restart nginx
```
# Resource:
certbot and nginx guide: https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04
certbot-dns-cloudflare’s documentation: 
https://certbot-dns-cloudflare.readthedocs.io/en/stable/index.html
https://joshuahidden.com/tutorials/installwildcard-ssl-with-cloudflare/
https://tcude.net/enabling-https-for-your-internal-nginx-instance/

nginx Debian install guide: https://nginx.org/en/linux_packages.html#Debian

Basic functions: https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/
admin guide -  security controls: https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/
web server - reverse proxy: https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
jellyfin and nginx config: https://jellyfin.org/docs/general/post-install/networking/reverse-proxy/nginx/

Some cool stuff too look at for nginx:
Basic Authentication: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
Mutual TLS (mTLS) authentication with OCSP:
- https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/#setup_ocsp
- https://docs.nginx.com/nginx-instance-manager/system-configuration/secure-traffic/#mutual-client-certificate-authentication-setup-mtls
- https://www.bastionxp.com/blog/configure-nginx-server-ssl-certificate-mutual-tls-client-authentication/
- https://dev.to/boobo94/how-to-setup-2way-ssl-authentication-mutual-authentication-with-nginx-3c1a
Nginx logs: https://www.digitalocean.com/community/tutorials/nginx-access-logs-error-logs

# Cool to know:
## The Anatomy of the Chain(ChatGPT):
To understand why, we have to look at how browsers trust certificates. Your server doesn't just need to prove who _it_ is; it needs to prove its "lineage" back to a trusted source.
1. **`cert.pem` (The Leaf):** This is your specific domain certificate. It contains your public key and your domain info.
2. **`chain.pem` (The Intermediates):** These are certificates from the Certificate Authority (CA) that bridge the gap between your specific certificate and the "Root" certificate built into your computer/phone.
3. **`fullchain.pem` (The Bundle):** This is simply `cert.pem` and `chain.pem` concatenated into one file.

