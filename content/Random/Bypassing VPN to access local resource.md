---
title: Bypassing VPN to access local resource
description:
draft: false
tags:
  - networking
  - VPN
created: 14/55/2025 11:55
updated: 23/50/2026 20:50
---
# Background:
When connecting a VPN on my host machine. I was not able to access my local servers as it is being routed through the VPN. My VPN provider does not have network based split tunnelling. 

# Configure:
To bypass VPN on the host. I use windows route command to manually add a route to tunnel all traffic that is pointing to my servers subnet to my ethernet gateway. That way only when accessing my local resource will it use my ethernet interface rather then tunnelling through the VPN.

Add persistent route to local subnet:
```powershell
route -p add <targeted_local_subnet_of_servers> MASK <subnet> <Gateway_IP> METRIC <METRIC VALUE>
```
Example: `route -p add 192.168.2.0 MASK 255.255.255.0 192.168.1.2 METRIC 5`

`METRIC` determines priority. the lower the value the more it's prioritized.  

View interfaces and verify routes:
```powershell
route print
```

Delete route:
```powershell
route delete <subnet>
```
Example: `route delete 192.168.2.0`

# How it works:
VPN works on the host by creating a Virtual Network Adapter. It then adds a entry on the routing table to tunnel all traffic through the VPN.

By adding a specific route with my server network subnet. it will take priority over the default gateway. 

side note that is cool to know:
The VPN route should typically have a lower cost metric then your network default. VPN route is also using the default gateway.
In the routing table. When there is conflicting routes. the one with the lower cost metric will take precedence. Thus the VPN interface will be the "new" default route. Cost metric is defined by the metric flag.

# Resources:
https://www.geeksforgeeks.org/how-to-add-a-static-route-to-windows-routing-table/
https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/route_ws2008
https://computer.howstuffworks.com/vpn.htm