---
title: Wazuh - Monitor docker containers in ubuntu
description:
draft: false
tags:
  - wazuh
  - SIEM
  - blue
  - defence
created: 25/41/2025 17:41
updated: 23/24/2026 21:24
---
# Installing required python librarys
In your machine with the agent installed. you need to install a few packages:
```bash
sudo apt install python3-docker python3-urllib3 python3-requests
```
Because of how ubuntu handles python packages. The command is different from the Wazuh documentation. 

# Enabling docker monitoring
Add the following data in the file `/var/ossec/etc/ossec.conf`.
```bash
sudo vim /var/ossec/etc/ossec.conf
```
```data
<wodle name="docker-listener">
    <disabled>no</disabled>
</wodle>
```
Put in anywhere within/inside of the `<ossec_config>` header.


Next restart the system. I found that restarting the Wazuh agent is not enough as when installing the python libraries. The comments mentions a system reboot is required to load the library.


Lastly in your Wazuh dashboard under `Cloud security` -> `Docker`, you should now see statistics related to your docker containers:
![[Wazuh - Monitor docker containers in ubuntu img 1.png]]

# Resources:
https://documentation.wazuh.com/current/user-manual/capabilities/container-security/monitoring-docker.html
