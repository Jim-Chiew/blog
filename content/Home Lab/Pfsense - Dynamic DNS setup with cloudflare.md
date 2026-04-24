---
title: Pfsense - Dynamic DNS setup with cloudflare
description:
draft: false
tags:
  - pfsense
  - dns
created: 21/29/2025 14:29
updated: 23/16/2026 21:16
---
Self hosted services like OpenVPN, Wireguard, Apache (selfhosted websites server) requires direct access to my house public IP in order for it to be accessible from the world wide web. but my house IP in Singapore is always changing. One solutions if to use dynamic DNS which links and updates my public IP to my domain that way I just need to remember my domain and it will direct me to my public IP address.

# Pre requisite:
You need a domain. You can subscribe to one with a Domain Registrars like Cloudflare, GoDaddy, Namecheap. There are free domains out there as well but will more limited domains names and top level domains to choose from.

# Setup Cloudflare A record.
An A record is a type of DNS record that links a domain name to an IPv4 address.
We need it to link our domain to our public IP. 

I am using Cloudflare. so in the Cloudflare dashboard: https://dash.cloudflare.com
Click on the domain you want to link to your public IP address:
![[Pfsense - Dynamic DNS setup img 1.png]]

Add an A record:
![[Pfsense - Dynamic DNS setup img 2.png]]
- The IP address can be set to anything for now. it will get replace with your IP later.
- The name will be your hostname. so in my case, I will access my public IP address with `DDNS.example.com`. 
- Proxy is disabled for me as I want to directly access my Pfsense. if you want to go through cloudflare proxy server first feel free to enable that. NOTE: some services do not handle proxy well so it is depended on what services you are trying to set up.

# Setup API key to allow modification of the DNS record:
You will need and API key in order to allow Pfsense to change the IP address of your DNS record.
In the Cloudflare dashboard ([link](https://dash.cloudflare.com/)). Click on `Manage accounts` -> `Account API Tokens` -> `Create Tokens`: 
![[Pfsense - Dynamic DNS setup img 3.png]]

Use the template titled `Edit Zone DNS`:
![[Pfsense - Dynamic DNS setup img 4.png]]

Enter the following details:
![[Pfsense - Dynamic DNS setup img 5.png]]
This will only grant the API key access to that specific domain.
Feel free to add a client IP address filtering if your Pfsense or client has a constant or predictable IP. Most ISP has a range of IP that you can insert if you want to do so.

Lastly click on `Continue to summary` -> `Create Token`. **Take note of the token as it will only be shown this one time**.

# Setup Dynamic DNS:
in your Pfsense dashboard. Go to `Services` -> `Dynamic DNS` -> `Add`:
![[Pfsense - Dynamic DNS setup img 6.png]]


Fill in your details:
![[Pfsense - Dynamic DNS setup img 7.png]]
Check IP modes determines how or when the IP is updated to the DNS. you can leave it at default as it will only use the check IP service if your WAN is assigned a private IP address. you can find out more about Check IP service [here](https://docs.netgate.com/pfsense/en/latest/services/dyndns/check-services.html).

Lastly fill in your credentials and click save:
![[Pfsense - Dynamic DNS setup img 8.png]]
Username leave it blank.
password is the API key that we created.
click save. 

Username should be used if you are using the legacy global API key. if that is the case use either you email address or your account ID which can be found in the right penal in you cloudflare dashboard under overview. 

There you are all setup:
![[Pfsense - Dynamic DNS setup img 9.png]]
You Cloudflare domain should reflect your public IP address.

# Resources:
https://docs.netgate.com/pfsense/en/latest/services/dyndns/client.html
https://www.wundertech.net/how-to-set-up-ddns-on-pfsense-using-cloudflare/
