---
title: Wazuh - Custom rules to alert on triggered pfsense rules
description:
draft: false
tags:
  - wazuh
  - SIEM
  - EDR
created: 04/57/2025 13:57
updated: 23/22/2026 21:22
---
#wazuh #SIEM #EDR
I have pfsense sending syslog to wazuh but it is lacking in description and alerts.  

# Creating custom rules:
Wazuh web interface: `Server management -> Rules -> Add new rules file`
example:

```xml
<!-- Custom pfsense rules. -->
<group name="custom_rules_example, pfsense,">
  <!-- 100100-100199 floating rules -->
  <rule id="100153" level="12">
    <if_sid>87700</if_sid>
    <id>1736860194</id>
    <description>VPN_client attampted to escape through WAN Interface.</description>
  </rule>
  
  <rule id="100112" level="12" frequency="2" timeframe="5" ignore="8">
    <if_matched_sid>87701</if_matched_sid>
    <same_id>1743734037</same_id>
    <same_srcip/>
    <same_dstip/>
    <same_dstport/>
    <description>Non-authorised cleints ($(srcip)) attemping to access firewall login page($(dstip) on port $(dstport)).</description>
  </rule>
</group>
```

# Breakdown Rule 1 (ID=100153):
- All rules need to be in at least one group.
- ID numbers between 100000 and 120000 for custom rules.

## rule attributes
contains 2 attributes in this example:
id - defines the ID of this rule
level - defines the level of the rule. Alerts and responses use this value.

## if_sid
We only want logs that are decoded as pf or pfsense logs. we don't want just any random logs to trigger it.

The id inside `if_sid` is defined as the rule that groups the logs. meaning in this case the id of `87700` points to another rule that labels logs decoded as `pf` to trigger it. 
![[Wazuh - Custom rules image2.png]]

## id
The next condition we want is to know which pfsense rule triggered it. by default the pfsense decoder in wazuh already decoded it and label the pfsense rule id as `id`.

The id is indicated at the bottom of the in the pfsense web portal rules as tracking ID:
![[Wazuh - Custom rules image3.png]]


# Breakdown of rule 2(100112)
```xml
 <rule id="100112" level="12" frequency="2" timeframe="5" ignore="8">
    <if_matched_sid>87701</if_matched_sid>
    <same_id>1743734037</same_id>
    <same_srcip/>
    <same_dstip/>
    <same_dstport/>
    <description>Non-authorised cleints ($(srcip)) attemping to access firewall login page($(dstip) on port $(dstport)).</description>
  </rule>
```
## Rule header
id and level is the same as previous.

frequency and timeframe works together to only trigger if the rule is triggered x amount of times(defined in frequency) within a certain time period(defined in timeframe).

ignore will ignore this rule for the next x amount of time defined in the ignore attribute.

in this case, only trigger if encountered 2 times within 5 sec and ignore the rest for 8 secs.

## Rule:
`if_matched_sid` Matches if an alert of the defined ID has been triggered in a set number of seconds. worked with `frequency` and `timeframe` rule header.

`same_id`, `same_srcip`, `same_dstip`, `same_dstport` works pretty much the same. it ensure that only those field have to be the same for the rule to be applied.

Meaning 2 different `srcip` for example would not be grouped as one but treated as 2 different instances. 

## Note: frequency minimum:
Frequency has a minimum value of 2.

# References:
[Rules Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)
[Rules classification (level)](https://documentation.wazuh.com/current/user-manual/ruleset/rules/rules-classification.html)

