---
title:
description:
draft: true
tags:
created: 28/03/2025 14:03
updated: 23/59/2026 21:59
---
I realized that some web crawler scrape networks by Ip addresses. So to minimized my footprint I found a way to configure Nginx to block/drop access through my IP address.

In `/etc/nginx/conf.d/default.conf`, add the following lines:
```bash
sudo vim /etc/nginx/conf.d/default.conf
```
```data
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  listen 443 default_server;
  listen [::]:443 default_server;

  ssl_reject_handshake on;

  server_name "";
  return 444;
}
```
`default_server` tag sets the server block to be the default for routes not matching any other server.

Restart nginx:
```bash
sudo systemctl restart nginx
```

# Resources:
https://stackoverflow.com/questions/29104943/how-to-disable-direct-access-to-a-web-site-by-ip-address
https://nginx.org/en/docs/http/request_processing.html
https://docs.nginx.com/nginx/admin-guide/web-server/web-server/#set-up-virtual-servers
https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_reject_handshake