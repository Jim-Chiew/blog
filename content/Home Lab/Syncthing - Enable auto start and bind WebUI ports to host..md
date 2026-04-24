---
title: Syncthing - Enable auto start and bind WebUI ports to host.
description:
draft: false
tags:
created: 13/40/2025 11:40
updated: 23/21/2026 21:21
---
Start Syncthing automatically on system start and bind it to host so you can access it from outside of the device by the device IP and its associated port: 

# Config auto start on boot: 
Using systemd approach:
Change myuser to your username or the user you want in charge of running synching. 
```bash
systemctl enable syncthing@myuser.service
systemctl start syncthing@myuser.service
```

# Changing default Web UI binds:
Get the config fil paths:
```bash
syncthing paths
```
```
/home/myuser/.local/state/syncthing/config.xml
```

Edit the config file in the user that is running syncthing. in my case it is `myuser`. use `su myuser` to be that user.
```bash
vim ~/.config/syncthing/config.xml
```
Modify the `address` value in the `gui` to `0.0.0.0:8384` to bind it to the host instead of loopback address: you should see something like:
```data
 <gui enabled="true" tls="true" debugging="false">
        <address>0.0.0.0:8384</address>
        <user>myuser</user>
        <password>password hash here</password>
        <apikey>api key here </apikey>
        <theme>default</theme>
 </gui>
```
Note: do not need to touch password and apikey. 
you can change the port `8284` to something else if you want.

```bash
sudo chown user:user ~/.config/syncthing/config.xml
```
# Resource: 
https://docs.syncthing.net/users/autostart.html#linux
https://docs.syncthing.net/users/config.html#config-option-device.remoteguiport

# Installing synching: 
https://apt.syncthing.net/

```bash
# Add the release PGP keys:
sudo mkdir -p /etc/apt/keyrings
sudo curl -L -o /etc/apt/keyrings/syncthing-archive-keyring.gpg https://syncthing.net/release-key.gpg
```

```bash
# Add the "stable-v2" channel to your APT sources:
echo "deb [signed-by=/etc/apt/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable-v2" | sudo tee /etc/apt/sources.list.d/syncthing.list
```

```bash
# Update and install syncthing:
sudo apt-get update
sudo apt-get install syncthing
```
