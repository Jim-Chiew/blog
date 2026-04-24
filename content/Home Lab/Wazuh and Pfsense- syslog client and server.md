---
title:
description:
draft: true
tags:
created: 03/53/2025 21:53
updated: 23/59/2026 20:59
---
#pfsense #wazuh #syslog

I want my Pfsense to send logs to Wazug. Wazuh will acts as a syslog server. Pfsense will act as a syslog client.

2 main steps:
1. Wazuh as syslog server
2. Pfsense as syslog client using Syslog-ng
# Wazuh as syslog server:
Enables Wazuh as a syslog server with:
- Port 514 for UDP
- Port 6514 for TCP

Edit the file `/var/ossec/etc/ossec.conf`. 
There will be a default `<remote>`, add the following line below the `<remote>` with:
```conf
 <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>udp</protocol>
    <allowed-ips>192.168.1.0/24</allowed-ips>
    <local_ip>192.168.10.10</local_ip>
  </remote>

  <remote>
    <connection>syslog</connection>
    <port>6514</port>
    <protocol>tcp</protocol>
    <allowed-ips>192.168.1.0/24</allowed-ips>
    <local_ip>192.168.10.10</local_ip>
  </remote>
```
The `<allowed-ips>` is required. 
The `<local_ip>` is the wazuh servers ip.

Restart Wazuh manager:
```bash
systemctl restart wazuh-manager
```

References:
https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/syslog.html

---
# Pfsense as syslog client using Syslog-ng:
Was not able to work with default Remote Logging in pfsense. Thus installed the package `Syslog-ng`.
Syslog-ng will act as a forwarder by listening on its local port and forwarding what it receives to wazuh. 

The following steps is taken:
1. Installation
2. Setup syslog listener
3. Forward logs to Syslog-ng
4. Syslog-ng forward to Wazuh

### 1. Installation:
Pfsense web interface: `System -> Package Manager -> Available Packages` search `Syslog-ng` and install.

### 2. Setup syslog  listener:
Pfsense web interface: `Services -> Syslog-ng -> General`:
My configuration:
![[Pfsense and Wazuh - syslog image1.png.png]]

### 3. Forward logs to Syslog-ng:
Pfsense web interface: `Status -> System Logs -> Settings -> Remote Logging Options`
My config
![[Pfsense and Wazuh - syslog image2.png.png]]
Ensure that the port in remote log server is the same as the one set in step 2.

### 4. Syslog-ng forward to Wazuh
Create 3 objects that does the following:
- Destination: Defines the IP of Wazug.
- Rewrite: Changes the log data.
- Log: Define what to do with logs.

At the end your Object should look something like this:
![[Pfsense and Wazuh - syslog image6.png.png]]

#### 4.1. Object - Destination:
Defines where wazuh server is.

Pfsense web browser: `Services -> Syslog-ng -> Advanced -> Add`
![[Pfsense and Wazuh - syslog image3.png.png]]

```
{ network("192.168.10.10" transport(tcp) port(6514)); };
```

Resource:
https://syslog-ng.github.io/admin-guide/070_Destinations/310_syslog-ng/000_syslog-ng_destination_options

#### 4.2. Object Rewrite:
Change hostname from localhost to pfsense.
![[Pfsense and Wazuh - syslog image4.png.png]]

```
{ set("pfsense", value("HOST"));};
```

Resource:
https://syslog-ng.github.io/admin-guide/110_Template_and_rewrite/001_Modifying_messages/001_Setting_fields

#### 4.3. Object Log:
Defines what to do with logs:
![[Pfsense and Wazuh - syslog image5.png.png]]
```
{ source(_DEFAULT); rewrite(REWRITE_HOSTNAME); destination(to_wazuh); };
```
Ensure that the attribute names are the same as the object names as defined earlier. 

# Done:
At this point in time you should be receiving log from pfsense.

