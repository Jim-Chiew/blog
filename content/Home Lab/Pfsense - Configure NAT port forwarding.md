---
title: Pfsense - Configure NAT port forwarding
description:
draft: false
tags:
  - pfsense
  - networking
created: 21/31/2025 18:31
updated: 23/15/2026 21:15
---
# Pre-requisite:
I have already set up an alias with geo restriction in Pfblocker.

# Setup:
Go to port forwarding and click add:
![[Pfsense - configure NAT port forwarding img 1.png]]

Here is my setup:
![[Pfsense - configure NAT port forwarding img 2.png]]
Some things to note:
NAT reflection - is used to determine how an internal devices will react to accessing the public IP of the forwarded rule. read more [here](https://docs.netgate.com/pfsense/en/latest/nat/reflection.html).
Filter rule association - when set to add associated filter rule, it will automatically add the rule to allow port forwarding on the firewall rule. if set to NON, you would have to manually add the firewall rule to allow incoming traffic.

# Resources:
https://docs.netgate.com/pfsense/en/latest/nat/port-forwards.html
https://docs.netgate.com/pfsense/en/latest/nat/reflection.html
https://docs.netgate.com/pfsense/en/latest/recipes/port-forwards-from-local-networks.html
https://www.youtube.com/watch?v=1YDVebJlGbM
