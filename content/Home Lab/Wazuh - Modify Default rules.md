---
title: Wazuh - Modify Default rules
description:
draft: false
tags:
  - wazuh
  - blue
  - defence
created: 25/36/2025 16:36
updated: 23/23/2026 21:23
---
The rule ID `510` in `0015-ossec_rules.xml` with the description of `Host-based anomaly detection event (rootcheck).` is just flooding the dashboard due to my LXC packages being on `/dev/`.

# Setup:
Wazuh allows you to modify its out-of-the-box rules by adding the `overwrite="yes"` on a custom rule.

Web Rule config:
```conf
<!-- 100000-100009    Custom overwrite -->

<group name="overwrite,">
  <rule id="510" level="7" overwrite="yes">
      <if_sid>509</if_sid>
      <description>Host-based anomaly detection event (rootcheck).</description>
      OS_Match
      <regex negate="yes" type="osmatch">.lxc</regex>
      <group>rootcheck,pci_dss_10.6.1,gdpr_IV_35.7.d,</group>
      <!-- <if_fts />  -->
  </rule>
</group>
```
Important thing is to ensure you use `overwrite="yes"` in the `rule` header tag.

# Resouce:
https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html#changing-existing-rules
https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#regex-rules
https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html

