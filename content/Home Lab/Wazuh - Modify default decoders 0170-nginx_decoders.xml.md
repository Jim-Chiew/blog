---
title: Wazuh - Modify default decoders 0170-nginx_decoders.xml
description:
draft: false
tags:
  - wazuh
  - blue
  - defence
created: 22/28/2025 18:28
updated: 23/23/2026 21:23
---
#wazuh #blue #defence

# 1. Copy decoder to custom folder location:
Default decoders is located in `/var/ossec/ruleset/decoders/` in your Wazuh server. Copy the file you want to modify to `/var/ossec/etc/decoders/`. This is to keep changes upon update of wazuh.

So in my case I want to modify `0015-ossec_rules.xml`:
```bash
sudo cp /var/ossec/ruleset/decoders/0170-nginx_decoders.xml /var/ossec/etc/decoders/
```

Change ownership:
```bash
sudo chown wazuh:wazuh 0170-nginx_decoders.xml
```

Assign permissions:
```bash
sudo chmod 660 0170-nginx_decoders.xml
```

# 2. Exclude the original in wazuh config file:
modify `/var/ossec/etc/ossec.conf` with the following:
```bash
sudo vim /var/ossec/etc/ossec.conf
```
```data
<ruleset>
  <!-- Default ruleset -->
  <decoder_dir>ruleset/decoders</decoder_dir>
  <rule_dir>ruleset/rules</rule_dir>
  <rule_exclude>0215-policy_rules.xml</rule_exclude>
  <list>etc/lists/audit-keys</list>

  <!-- User-defined ruleset -->
  <decoder_dir>etc/decoders</decoder_dir>
  <rule_dir>etc/rules</rule_dir>

  <!-- Custom exclude -->
  <decoder_exclude>ruleset/decoders/0170-nginx_decoders.xml</decoder_exclude>
</ruleset>
```

# Restart wazuh:
```bash
systemctl restart wazuh-manager
```

From here you can edit the decoder for your needs by modifying the `0170-nginx_decoders.xml` file in the Wazuh web interface 