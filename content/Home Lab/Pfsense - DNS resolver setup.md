---
title: Pfsense - DNS resolver setup
description:
draft: false
tags:
  - dns
  - networking
  - pfsense
created: 17/42/2025 19:42
updated: 23/15/2026 21:15
---
# Setup DNS for pfsense
Under the main webpage go to: System -> General Setup.
I will be using Cloudflare as my DNS of choice:

![[Pfsense - DNS resolver setup img 1.png]]
You can find out more about cloudflare DNS here:
https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-tls/

# Set up DNS resolver:
Here are my configurations:
![[Pfsense - DNS resolver setup img 2.png]]

Notes: 
I have created a seperate SSL/TLS cert just for the DNS resolver.
I only want my WLAN, LAN, and server to use the DNS resolver.
Enabled DNSSEC, DNS quarry forwarding and SSL/TLS for outgoing DNS queries.

# Resource: 
https://docs.netgate.com/pfsense/en/latest/recipes/dns-over-tls.html
