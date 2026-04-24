---
title: making a user cheatsheet
description:
draft: false
tags:
  - Cheatsheet
  - Linux
created: 23/40/2025 11:40
updated: 23/09/2026 21:09
---
# Make the user: 
```bash
sudo adduser username
```
```bash
adduser username
```

# Creating a home directory: 
```bash
sudo mkhomedir_helper username
```
```bash
mkhomedir_helper username
```

# Add user to sudo:
```bash
sudo usermod -aG sudo username
```
```bash
usermod -aG sudo username
```

# Disabling root:
```bash
sudo passwd -l root
```
Check:
```bash
sudo cat /etc/shadow | grep root
```
Ensure that `root:` is followed by `!`.

# Modify user shell:
```bash
sudo usermod -s /usr/sbin/nologin root
```

## Resource:
https://itsfoss.gitlab.io/post/4-ways-to-disable-root-account-in-linux/

# No login system User:
```bash
sudo adduser --system --shell /usr/sbin/nologin userName
```

## Resource:
https://www.baeldung.com/linux/create-non-login-user#bd-1-creating-anologin-system-user